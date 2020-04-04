---
layout: post
title: 如何使用Line Bot整合CI/CD流程 - 開發篇 Message API
categories: [I.T.]
date: 2020-04-04 05:00:00
tags: [A.W.S,I.T.,CI/CD]
---
最近在為公司導入CI/CD流程，需要建構開發、測試、正式環境滿足業務需求，由於公司運行的架構採用的是A.W.S的Lambda Serverless環境運行，開發人員首先利用A.W.S Lambda開發環境進行線上開發，當開發人員開發完成，使用Postman透過A.W.S. API Gateway做測試，完成後便發行新的Lambda版本，使用Lambda的Alias Name分別劃分開發、測試、正式環境，並對版本產生連結。在這樣的情境下，當發行新Lambda版本或更新，若牽涉到的Lambda Function數過多，往往造成部署上的麻煩，因此開發Line Bot將佈署整合程序寫入Lambda，透過呼叫Lambda Function使用Line Bot做控制，針對開發過程中所涉及的角色功能做權限上的劃分。

<!--more-->

本篇是以Line Message API做為對接的A.W.S Lambda function Code ，關於Demo功能介紹請查看下列文章。

Demo篇

[如何使用Line Bot整合CI/CD流程 - Admin Demo]({% post_url 2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin %})

[如何使用Line Bot整合CI/CD流程 - dev Demo]({% post_url 2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev %})

[如何使用Line Bot整合CI/CD流程 - Test Demo]({% post_url 2020-04-02-how-to-integrate-ci-cd-process-with-line-bot-demo-tester %})

<hr>

本篇所提供的Lambda Function Code主要做為Line Message API對接的API（以下皆稱Message API），當Line聊天室機器人收到任何一則文字訊息或Post Back訊息，Line聊天室機器人會將相關資訊傳遞到這個Message API做處理。在本次專案實驗過程中，這隻Message API如若接到關於CI/CD相關命令，則會呼叫相對應的CI/CD Lambda Function，以收到要檢測Lambda版本的命令為例，則會呼叫GETSTAGEDIFFLISTARN該隻ARN的CI/CD Lambda Function做處理，待該CI/CD Lambda Function處理完成後，便會回傳到本隻Message API，並且回傳使用者所需要的資訊到使用者的Line上。

要注意的是，在本次專案實驗過程中，僅單純使if..else作為權限管理的判斷，如若需要更複雜的權限管理功能，需要額外的實作，以下提供該隻Message API的程式碼:

提醒：
1. 我將關於隱私的相關資訊全部都儲存在Message API當中，如若要取得相關變數，則需透過 os.environ['KEY'] 這樣的API做實現。
2. 我將關於CI/CD的程式碼佈署在主要功能外的地區，由於Message API存取主要功能皆是透過其他CI/CD Lambda Function，因此您將在下列程式碼中看不到關於地區變數的選項，如若需要可再依您的情境修改。
3. 關於部署紀錄的資料庫我將會額外在開一篇 如何使用Line Bot整合CI/CD流程 - 資料庫篇 做介紹。
4. Line的replyToken只能回傳一次訊息，因此如果一個功能需要回傳兩個非即時性的訊息則要以一個reply和一個push做傳遞。
5. 程式碼第221列，由於Line PostBack事件的傳遞資料有字數長度限制，因此僅傳遞必要資訊，如PostBack資料裡僅有deployID，程式即可知道這是一個部署事件，若有deployEnv，即可知道是查詢環境部署清單事件，若為退版事件，需要有rollBackID，而rollBackID為部署完成後回傳的DeployID。
6. 查詢環境版本差異的CI/CD Lambda將在其他篇中介紹。

```python
#import相關library
import  requests
import  json
import  os
import boto3
import time
import pymysql

#將所有需要的常數變數在Lambda initialize時便設置好
LINEACCESSTOKEN:str = os.environ['LINEACCESSTOKEN']
DBNAME:str = os.environ['DBNAME']
DBUSER:str = os.environ['DBUSER']
DBHOST:str = os.environ['DBHOST']
DBPASSWORD:str = os.environ['DBPASSWORD']
ADMINLINEUSERID:str = os.environ['ADMINLINEUSERID']
DEPLOYSTAGEARN:str = os.environ['DEPLOYSTAGEARN']
GETSTAGEDIFFLISTARN:str = os.environ['GETSTAGEDIFFLISTARN']

#Line Message API呼叫時所需要的Header在Lambda initialize時便設置好
HEADER = {
    'Content-type': 'application/json',
    'Authorization': 'Bearer ' + LINEACCESSTOKEN
}

#Line Message API reply function介面
def replyMessage(payload):
    response = requests.post('https://api.line.me/v2/bot/message/reply',headers=HEADER,data=json.dumps(payload))
    print(response.text)

#Line Message API push function介面
def pushMessage(payload):
    response = requests.post('https://api.line.me/v2/bot/message/push',headers=HEADER,data=json.dumps(payload))
    print(response.text)

#取得Line的User Profile
def getUserProfile(userID):
    response = requests.get('https://api.line.me/v2/bot/profile/'+str(userID),headers=HEADER)
    return json.loads(response.text)

#取得Line的User Name
def getUserName(userID):
    return getUserProfile(userID)["displayName"]

#呼叫Lambda Function的共通接口，需帶入Lambda Func的Arn，要傳遞的參數bodyData
def invokeLambdaFunc(arn:str, bodyData:dict):
    client = boto3.client('lambda')
    response = client.invoke(FunctionName=arn,
                                 InvocationType='RequestResponse',
                                 Payload=json.dumps(bodyData))
    resObj =  json.loads(json.loads(response['Payload'].read().decode('utf-8'))["body"])
    return resObj

#當有部署事件發生時，需到資料庫記錄，deployID通常預設為TimeStamp
def insertDeployRecord(deployID,deployEnv, insertData,deployer):
    try:
        conn = pymysql.Connection(host=DBHOST, database=DBNAME,
            user=DBUSER, password=DBPASSWORD)
        cur = conn.cursor()
        sql = "INSERT INTO DeployRecord(DeployID, DeployType, FunctionName,OriginVersion,TargetVersion,Deployer,isDeploy) values(%s, %s, %s, %s, %s, %s, %s)"
        batchData = list()
        for iData in insertData:
            dataTuple = (deployID, deployEnv, iData["fName"],iData["originVersion"],iData["targetVersion"], deployer, False)
            batchData.append(dataTuple)
        tmp = tuple(batchData)
        cur.executemany(sql, tmp)
        conn.commit()
    except Exception as e:
        print(e)
    finally:
        cur.close()
        conn.close()
    return 1

#更新部署狀態，若不帶入狀態預設為設置已部署
def updateDeployRecord(deployID,isDeploy=1):
    try:
        conn = pymysql.Connection(host=DBHOST, database=DBNAME,
            user=DBUSER, password=DBPASSWORD)
        cur = conn.cursor()
        sql = "UPDATE DeployRecord SET isDeploy=%s WHERE DeployID=%s;"% (isDeploy, deployID)
        cur.execute(sql)
        conn.commit()
    except Exception as e:
        print(e)
    finally:
        cur.close()
        conn.close()
    return 1

#取得部署狀態資料
def getDeployRecord(deployID, isDeploy=0):
    retList = list()
    try:
        conn = pymysql.Connection(host=DBHOST, database=DBNAME,
            user=DBUSER, password=DBPASSWORD)
        cur = conn.cursor()
        sql = "SELECT DeployID, DeployType, FunctionName,OriginVersion,TargetVersion FROM DeployRecord WHERE DeployID = %s AND IsDeploy = %s;" % (deployID, isDeploy)
        cur.execute(sql)
        rows = cur.fetchall()
        for row in rows:
            retList.append({"deployID":row[0],"deployType":row[1],"fName":row[2],"originVersion":row[3], "targetVersion":row[4]})
    except Exception as e:
        print(e)
    finally:
        cur.close()
        conn.close()
    return retList

#若要部署測試環境，則回調是由dev部署到test，若要部署正式環境，則回調是由test部署到prod
def getQualifier(deployEnv):
    return ("dev","test") if deployEnv == "test" else ("test","prod")

#取得最新測試環境及正式環境在資料庫中的資料
def getLastDeployRecord(isDeploy=1):
    retList = list()
    try:
        conn = pymysql.Connection(host=DBHOST, database=DBNAME,
            user=DBUSER, password=DBPASSWORD)
        cur = conn.cursor()
        sql = "SELECT MAX(DeployID) as DeployID, DeployType, Deployer FROM DeployRecord WHERE IsDeploy=%s GROUP BY DeployType ORDER BY DeployID ASC LIMIT 10;"%(isDeploy)
        cur.execute(sql)
        rows = cur.fetchall()
        for row in rows:
            retList.append({"deployID":row[0],"deployType":row[1],"deployer":row[2]})
    except Exception as e:
        print(e)
    finally:
        cur.close()
        conn.close()
    return retList

#訊息處理程式本體
def lambda_handler(event, context):
    body = dict()
    runCmd:str = ""
    replyToken:str = ""
    userToken:str = ""
    if "body" in event and event["body"]:
        body = json.loads(event['body'])
        #在此可以判斷是否由Line Message所傳遞的Data
        if "events" in body and len(body["events"])>0:
            #初始化replyToken
            if "replyToken" in body["events"][0]:
                replyToken = body["events"][0]["replyToken"]
            if "source" in body["events"][0]:
                userToken = body["events"][0]["source"]["userId"]
                #初始化Line UserID
                if userToken != ADMINLINEUSERID:
                    #非白名單不可操作
                    return -1
                #初始化User Name
                userName = getUserName(userToken)
            #判斷是否為PostBack Event
            if "postback" in body["events"][0]:
                if "data" in body["events"][0]["postback"]:
                    #取得PostBack Data
                    if "deployEnv" in body["events"][0]["postback"]["data"]:
                        if "&" in body["events"][0]["postback"]["data"]:
                            rawDataList:str = body["events"][0]["postback"]["data"].split("&")
                            deployEnvRaw:str = rawDataList[0]
                            rollBackRaw:str = rawDataList[1]
                        else:
                            deployEnvRaw:str = body["events"][0]["postback"]["data"]
                        deployEnv:str = deployEnvRaw.split("=")[1]
                        fQualifier = getQualifier(deployEnv)
                        #如若PostBack Data包含deployEnv的Key，可判斷為要取得部署資料，或要執行退版
                        if deployEnv in ("test","prod"):
                            runCmd = "getEnvData"
                        elif deployEnv == "rollBack":
                            rollBackID = int(rollBackRaw.split("=")[1])
                            runCmd = deployEnv
                    elif "deployID" in body["events"][0]["postback"]["data"]:
                        #如若PostBack Data包含deployID的Key，可判斷為要執行部署
                        deployID = int(body["events"][0]["postback"]["data"].split("=")[1])
                        runCmd = "deploy"
            elif "message" in body["events"][0]:
                #初始化檢測是否為預留的Command命令
                runCmd:str = body["events"][0]["message"]["text"]

        else:
            #檢測傳送單一訊息
            runCmd = "sendOneTextMessage"

    if runCmd == "sendOneTextMessage":
        #傳送單一訊息
        payload = {
            'to': ADMINLINEUSERID,
            'messages': [
                {
                "type":"text",
                "text":body['message']
                }
                ]
        }
    elif runCmd == "getEnvData":
        #傳送環境部署清單
        payload = {
                'replyToken': replyToken,
                'messages':[{"type":"text","text":"正在取得部署清單..."}]
                  }
        #讓使用者知道Message API確實有收到指令
        replyMessage(payload)
        retBody=dict()
        retBody["messages"] = list()
        #取得環境版本差異
        resultList = invokeLambdaFunc(arn=GETSTAGEDIFFLISTARN, bodyData={"deployEnv":deployEnv})
        messageText:str = "偵測到下列功能"+fQualifier[0]+"環境與"+fQualifier[1]+"環境版本不一致，是否進行部署？"
        tmpStr = ""
        for r in resultList:
            tmpStr+=r["fName"]+"\n"

        if len(resultList):
            messageText = messageText + "\n" + tmpStr.strip()
        else:
            messageText = ""

        if len(messageText)>0:
            #預設的ID為TimeStamp
            deployID:int = int(time.time())
            #記錄可部署的資料
            insertDeployRecord(deployID,deployEnv, resultList, userName)
            retBody["messages"].append(
            {
                    "type":"text",
                    "text": messageText
            })
            retBody["messages"].append(
                {
                          "type": "template",
                          "altText": "this is a confirm template",
                          "template": {
                              "type": "confirm",
                              "text": "確認是否部署？",
                              "actions": [
                                  {
                                   "type":"postback",
                                   "label":"Deploy",
                                   "data":"deployID="+str(deployID)
                                  },
                                  {
                                    "type": "message",
                                    "label": "No",
                                    "text": "no"
                                  }
                              ]
                          }
                })
            body = retBody
        else:
            #無版本可更新
            body = dict()
            body["messages"] = list()
            body["messages"].append(
            {
                    "type":"text",
                    "text": "沒有新的版本需要更新！"
            })
        payload = {
                'to': userToken,
                'messages': body["messages"]
            }
        pushMessage(payload)
    elif runCmd == "deploy":
        #部署事件
        payload = {
                'replyToken': replyToken,
                'messages':[{"type":"text","text":"正在開始部署..."}]
                  }
        #傳遞訊息讓使用者確認程式有收到部署事件
        replyMessage(payload)
        #透過deployID取得要部署的Function及版本
        deployList = getDeployRecord(deployID)
        #執行部署
        invokeResult = invokeLambdaFunc(arn = DEPLOYSTAGEARN, bodyData = deployList)
        print("invokeResult:",invokeResult)
        messageList = list()
        if invokeResult["message"]=="done":
            updateDeployRecord(deployID)
            messageList.append(
            {
                "type":"text",
                "text":"DeployID:"+str(deployID)+"部署完成"
            })

        payload = {
                'to': userToken,
                'messages': messageList
            }
        #回傳部署完成訊息
        pushMessage(payload)
    elif runCmd == "環境佈署清單":
        #取得環境部署清單
        if userToken != ADMINLINEUSERID:
            #非白名單內無法取得該名單
            payload = {
                'replyToken': replyToken,
                'messages':[{"type":"text","text":"抱歉您沒有足夠的權限查看環境佈署清單"}]
                  }
            replyMessage(payload)
            return -1
        #環境部署樣板
        deployTemplate= [{
                          "type": "template",
                          "altText": "This is a buttons template",
                          "template": {
                                      "type": "buttons",
                                      "title": "環境佈署清單",
                                      "text": "請選擇",
                                      "actions": [
                                          {
                                            "type": "postback",
                                            "label": "開發/測試環境",
                                            "data": "deployEnv=test"
                                          },
                                          {
                                            "type": "postback",
                                            "label": "測試/正式環境",
                                            "data": "deployEnv=prod"
                                          }
                                      ]
                                     }
                        }]
        payload = {
                'replyToken': replyToken,
                'messages':deployTemplate
                  }
        replyMessage(payload)
    elif runCmd == "退版清單":
        #取得可退版清單
        #可退版的條件為要已經部署完成的才可執行退版
        lastDeployList = getLastDeployRecord()
        actionList = list()
        #生成退版清單
        for ld in lastDeployList:
            deployID:str = str(ld["deployID"])
            actionList.append(
                {
                    "type": "postback",
                    "label": ld["deployType"]+":"+ deployID,
                    "data": "deployEnv=rollBack&rollBackID=" + deployID
                })
        deployTemplate= [{
                          "type": "template",
                          "altText": "This is a buttons template",
                          "template": {
                                      "type": "buttons",
                                      "title": "退版清單",
                                      "text": "請選擇",
                                      "actions": actionList
                                     }
                        }]
        payload = {
                'replyToken': replyToken,
                'messages':deployTemplate
                  }
        replyMessage(payload)
    elif runCmd == "環境最新佈署人員":
        #取得環境最新佈署人員名單
        retText = ""
        lastDeployList = getLastDeployRecord()
        for lastDeploy in lastDeployList:
            #取得佈署類型
            qualifier = getQualifier(lastDeploy["deployType"])
            retText += "最後一次由%s環境佈署至%s環境，是由%s佈署，DeployID是：%s\n"% (qualifier[0],qualifier[1],lastDeploy["deployer"],lastDeploy["deployID"])
        payload = {
                'replyToken': replyToken,
                'messages':[{"type":"text","text":retText.strip()}]
                  }
        replyMessage(payload)
    elif runCmd == "rollBack":
        #執行退版命令
        payload = {
                'replyToken': replyToken,
                'messages':[{"type":"text","text":"開始退版程序..."}]
                  }
        #傳遞訊息讓使用者確認程式有收到退版事件
        replyMessage(payload)
        print("rollBackID:",rollBackID)
        #postback data中傳遞的rollBackID就是當時完成deployID
        rollBackList = getDeployRecord(deployID=rollBackID,isDeploy=1)
        tempList = list()
        #修改當時的部署env作為退版依據
        for rb in rollBackList:
            tempDict = rb
            tempDict["deployType"] = runCmd + "-" + rb["deployType"]
            tempList.append(tempDict)
        #交由執行部署的function完成
        invokeResult = invokeLambdaFunc(arn = DEPLOYSTAGEARN, bodyData = tempList)
        print("invokeResult:",invokeResult)
        #更新已部署完成的記錄退回未部署
        updateDeployRecord(deployID=rollBackID,isDeploy=0)
        messageList = list()
        messageList.append(
            {
                "type":"text",
                "text":"DeployID:"+str(rollBackID)+"退版完成"
            })

        payload = {
                'to': userToken,
                'messages': messageList
            }
        pushMessage(payload)
    return {
        'statusCode': 200,
        'body': json.dumps({'work':'done'})
    }
```

其中要注意，由於預設的Lambda Function執行時間只有3秒，但在Function數量很大量時，查詢或部署時間絕對會超過3秒，因此早在完成時Message API早就因為執行過久而shutdown掉，因此要加大Message API Lambda Function的執行時間，本次實驗專案執行時間為十分鐘。

這邊額外說明一下當這隻Message API收到 Line訊息時，Event會呈現的格式
文字訊息：
```json
{"AWS HTTP KEY":"AWS HTTP VALUE", "body":"{'events':[{'type':'message','replyToken':'LINE REPLYTOKEN','source':{'userId':'LINE USERID','type':'user'},'timestamp':132456576,'mode':'active','message':{'type':'text','id':'1234567890','text':'環境佈署清單'}}],'destination':'XXXXXXXXXXX'}", "isBase64Encoded": false}
```
因此要取得文字訊息時須先將Body轉成json(172-173列)，取得
```json
{"events":[{"type":"message","replyToken":"LINE REPLYTOKEN","source":{"userId":"LINE USERID","type":"user"},"timestamp":132456576,"mode":"active","message":{"type":"text","id":"1234567890","text":"環境佈署清單"}}],"destination":"XXXXXXXXXXX"}
```
再以 body["events"][0]["message"]["text"] 方式取得文字內容。

postBack Event:
```json
{"AWS HTTP KEY":"AWS HTTP VALUE", "body":"{'events':[{'type':'message','replyToken':'LINE REPLYTOKEN','source':{'userId':'LINE USERID','type':'user'},'timestamp':132456576,'mode':'active','postback':{'data':'deployID=1585816272'}}],'destination':'XXXXXXXXXXX'}'", "isBase64Encoded": false}
```
因此要取得文字訊息時須先將Body轉成json(172-173列)，取得
```json
{"events":[{"type":"postback","replyToken":"LINE REPLYTOKEN","source":{"userId":"LINE USERID","type":"user"},"timestamp":132456576,"mode":"active","postback":{"data":"deployID=1585816272"}}],"destination":"XXXXXXXXXXX"}

```
再以 body["events"][0]["postback"]["data"] 方式取得文字內容。

另外，由於Message API在執行時需Invoke其他的CI/CD的Lambda Function，因此在Message API Lambda執行的Role上，除了基本的權限，還需額外賦予 lambda:InvokeFunction 的權限，該Policy最終會如下所示。
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "lambda:InvokeFunction",
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:us-east-1:XXXXXXXXXXXXX:log-group:/aws/lambda/MessageAPI:*"
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "arn:aws:logs:us-east-1:XXXXXXXXXXXXX:*"
        }
    ]
}
```

Message API相關設定及程式碼到這邊介紹完畢。
