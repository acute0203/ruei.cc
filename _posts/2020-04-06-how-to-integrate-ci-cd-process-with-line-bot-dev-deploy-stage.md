---
layout: post
title: 如何使用Line Bot整合CI/CD流程 - 開發篇 部署環境
categories: [I.T.]
date: 2020-04-06 00:00:00
tags: [A.W.S,I.T.,CI/CD]
---
最近在為公司導入CI/CD流程，需要建構開發、測試、正式環境滿足業務需求，由於公司運行的架構採用的是A.W.S的Lambda Serverless環境運行，開發人員首先利用A.W.S Lambda開發環境進行線上開發，當開發人員開發完成，使用Postman透過A.W.S. API Gateway做測試，完成後便發行新的Lambda版本，使用Lambda的Alias Name分別劃分開發、測試、正式環境，並對版本產生連結。在這樣的情境下，當發行新Lambda版本或更新，若牽涉到的Lambda Function數過多，往往造成部署上的麻煩，因此開發Line Bot將佈署整合程序寫入Lambda，透過呼叫Lambda Function使用Line Bot做控制，針對開發過程中所涉及的角色功能做權限上的劃分。

<!--more-->

本篇是部署功能Lambda Function版本功能作為本篇介紹，關於本系列其他介紹請查看下列文章。

Demo篇

[如何使用Line Bot整合CI/CD流程 - Admin Demo]({% post_url 2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin %})

[如何使用Line Bot整合CI/CD流程 - dev Demo]({% post_url 2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev %})

[如何使用Line Bot整合CI/CD流程 - Test Demo]({% post_url 2020-04-02-how-to-integrate-ci-cd-process-with-line-bot-demo-tester %})

開發篇

[如何使用Line Bot整合CI/CD流程 - 開發篇 Message API]({% post_url 2020-04-04-how-to-integrate-ci-cd-process-with-line-bot-dev-message-api %})

[如何使用Line Bot整合CI/CD流程 - 開發篇 取得環境差異名單]({% post_url 2020-04-05-how-to-integrate-ci-cd-process-with-line-bot-dev-stage-diff %})

Line篇

[如何使用Line Bot整合CI/CD流程 - Line篇 介面設定]({% post_url 2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface %})

<hr>

本篇所提供的Lambda Function Code主要做為Message API的對接，其主要功能為部署所提供的Function資訊對該Function做部署，而部署方式可分為三種，第一種為將開發環境的程式碼部署到測試環境，由於開發環境與測試環境的差異，開發環境使用的程式碼為較新的程式碼，可以發佈新的版本，關於這樣的設置，值得注意的有兩點，第一點是經由發佈後的新版本，新版本的程式碼無法做變更，第二點是因應第一點而生，因此需要將相關的組態設定作為環境變數帶入不同的環境，必須說這樣的設計方式是正確且值得讚賞的，因為沒有道理說因為不同的環境而對程式碼做不同的變更。

第二種為將測試環境的程式碼部署到正式環境，這樣的部署方式較為單純，僅需將測試環境的版本作為正式環境的變數更新即可。

第三種為退版功能，需將原始紀錄的版本資訊作為變數，帶入被部署的環境。

程式碼如下：

```python
#import需要的Library
import json
import boto3

#程式處理本體
def lambda_handler(event, context):
    # TODO implement
    #boto3初始化，並將地區參數指定在要部署Lambda Function的地區
    client = boto3.client('lambda', region_name='us-west-2')
    #輸入參數即為要部署的Function List和所需的相關資訊
    deployList = event
    for deployFunc in deployList:
        fName = deployFunc["fName"]
        deployEnv = deployFunc["deployType"]
        originVersion = deployFunc["originVersion"]
        targetVersion = deployFunc["targetVersion"]

        if deployEnv == "test":
            #當要部署的環境為測試環境，需要在開發環境中發佈新的版本
            print(fName + " have update the function.")
            pvResult = client.publish_version(FunctionName = fName)
            #記錄新發行的版本
            newVersion = pvResult["Version"]
            print(fName + " publish new version:",newVersion)
        elif deployEnv == "prod":
            #當要部署的環境為正式環境，僅需要取得測試環境的版本即可
            newVersion = originVersion
            print("newVersion:",newVersion)
        elif deployEnv.startswith("rollBack"):
            #當要退回版本，須先取得要退回的是處於什麼環境的版本
            rollBackEnv = deployEnv.split("-")[1]
            deployEnv = rollBackEnv
            #當時紀錄targetVersion的版本即為前一版的版本
            newVersion = targetVersion
            print("Function:%s, rollBack to Version:%s"%(fName, newVersion))
        updateResult = client.update_alias(FunctionName=fName, Name=deployEnv, FunctionVersion=newVersion)
        #更新Alias環境的版本
    return {
        'statusCode': 200,
        'body': json.dumps({"message":"done"})
    }
```

其中要注意，由於預設的Lambda Function執行時間只有3秒，但在Function數量很大量時，查詢時間絕對會超過3秒，因此要加大部署環境 Lambda Function的執行時間，本次實驗專案執行時間為十分鐘。

另外，由於部署環境時，Lambda需要有額外的權限，第一個權限是當開發環境部署到測試環境時，要有發行新Lambda版本的權限，名稱為 lambda:PublishVersion ，第二個為無論部署到任何環境，都需要更新Alias所連結版本的權限，名稱為 lambda:UpdateAlias ，該Policy最終會如下所示。

Tips:
當我們在配適合適的IAM權限給該執行程式的Role時，有時候會因為IAM系統太過於複雜，所以乾脆直接賦予Full Access權限，這樣在開發上雖然方便，但反而會因為權限開得太大而有不恰當的操作，開發人員或資安人員應當以配適最小的權限為原則進行配適，在這裡我提供一個方法，當不知道要搭配什麼樣的權限給當前的Role時，不彷乾脆讓程式噴Error Log，觀察Log中顯示的權限不足來進行權限配置，第二個做法是查詢[Boto 3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)，可以觀察所調用的Function Name來進行權限或資源上的配置。以本專案為例，所用到的Function Name為[PublishVersion](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/lambda.html#Lambda.Client.publish_version)和[UpdateAlias](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/lambda.html#Lambda.Client.update_alias)，因此直接賦予該Function所需的權限（名稱一樣）。

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "lambda:PublishVersion",
                "lambda:UpdateAlias"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:us-east-1:xxxxxxxxxxxxxxx:log-group:/aws/lambda/deploy-stage:*"
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "arn:aws:logs:us-east-1:xxxxxxxxxxxxxxxx:*"
        }
    ]
}
```

部署環境相關設定及程式碼到這邊介紹完畢。

Reference: [Boto 3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
