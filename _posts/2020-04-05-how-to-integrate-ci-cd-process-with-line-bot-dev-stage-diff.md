---
layout: post
title: 如何使用Line Bot整合CI/CD流程 - 開發篇 取得環境差異名單
categories: [I.T.]
date: 2020-04-05 00:00:00
tags: [A.W.S,I.T.,CI/CD]
---
本篇是查詢Lambda Function版本功能作為本篇介紹，關於本系列其他介紹請查看下列文章。

<!--more-->

總覽

[如何使用Line Bot整合CI/CD流程 - Overview]({% post_url 2020-04-09-how-to-integrate-ci-cd-process-with-line-bot-overview %})

Demo篇

[如何使用Line Bot整合CI/CD流程 - Admin Demo]({% post_url 2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin %})

[如何使用Line Bot整合CI/CD流程 - dev Demo]({% post_url 2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev %})

[如何使用Line Bot整合CI/CD流程 - Test Demo]({% post_url 2020-04-02-how-to-integrate-ci-cd-process-with-line-bot-demo-tester %})

開發篇

[如何使用Line Bot整合CI/CD流程 - 開發篇 Message API]({% post_url 2020-04-04-how-to-integrate-ci-cd-process-with-line-bot-dev-message-api %})

[如何使用Line Bot整合CI/CD流程 - 開發篇 部署環境]({% post_url 2020-04-06-how-to-integrate-ci-cd-process-with-line-bot-dev-deploy-stage %})

Line篇

[如何使用Line Bot整合CI/CD流程 - Line篇 介面設定]({% post_url 2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface %})

資料庫篇

[如何使用Line Bot整合CI/CD流程 - 資料庫]({% post_url 2020-04-08-how-to-integrate-ci-cd-process-with-line-bot-database %})

<hr>

本篇所提供的Lambda Function Code主要做為Message API的對接，其主要功能為查詢目標地區的Lambda Function程式碼經過Hash計算後，有哪些是不同的，因為相同的Hash Code背後隱含的是該程式碼已經經過部署，並且經過部署後，被部署的環境和部署環境並沒有差異，反之則是被部署的環境和部署環境有差異，因此需要重新部署，舉個簡單的例子來看，假設我們要將開發環境的程式碼部署到測試環境，要部署的原因是因為開發環境有新的程式碼版本所以才需要經過測試人員檢測，因此開發環境新的程式碼和測試環境的程式碼一定會不同，進而經過Hash運算後，而有不同的結果，因此我們可以得知，當兩個程式碼Hash結果若不同，則一定有新的版本需要新的部署。

程式碼如下

```python
#import需要的Library
import json
import boto3

#程式處理本體
def lambda_handler(event, context):
    # TODO implement
    #從帶入的參數中取得要將程式碼部署到哪個環境
    deployEnv = event["deployEnv"]
    resultList = list()
    #boto3初始化，並將地區參數指定在要偵測Lambda Function的地區
    client = boto3.client('lambda', region_name='us-west-2')
    #取得該地區的Lambda Func清單
    lambdaFunctions = client.list_functions()["Functions"]
    for lf in lambdaFunctions:
        fName = lf["FunctionName"]
        fArn = lf["FunctionArn"]
        #初始化取得所需比較環境的Alias Name
        if deployEnv == "test":
            fQualifier = ("dev","test")
        elif deployEnv == "prod":
            fQualifier = ("test","prod")
        fSha256 = lf["CodeSha256"]

        try:
            #取得被部署環境的Function
            originConfiguration:str = client.get_function(FunctionName=fName,Qualifier=fQualifier[0])["Configuration"]
             #取得被部署環境的Function Hash code
            originSha256:str = originConfiguration["CodeSha256"]
            #取得部署環境的Function
            targetConfiguration:str = client.get_function(FunctionName=fName,Qualifier=fQualifier[1])["Configuration"]
             #取得部署環境的Function Hash code
            targetSha256:str = targetConfiguration["CodeSha256"]
            #檢測使用的Layer或其版本是否有差異
            originLayerSet = set()
            targetLayerSet = set()
            for oLayer in originConfiguration["Layers"]:
                originLayerSet.add(oLayer["Arn"])
            for tLayer in targetConfiguration["Layers"]:
                targetLayerSet.add(tLayer["Arn"])
            LayerDiff = originLayerSet.symmetric_difference(targetLayerSet)
            #若兩環境的Hash Code不同或Layer有差異，則代表該Function偵測到有新的版本
            if originSha256 != targetSha256 or len(LayerDiff) > 0:
                #加入回傳的List
                resultList.append({"fName":fName,"originVersion":originConfiguration["Version"],"targetVersion":targetConfiguration["Version"]})
                #print(fName+"'s "+ fQualifier[1]+ " enviroment" + " have update to version:" + updateResult["FunctionVersion"])
        except Exception as e:
            pass

    return {
        'statusCode': 200,
        'body': json.dumps(resultList)
    }

```

其中要注意，由於預設的Lambda Function執行時間只有3秒，但在Function數量很大量時，查詢時間絕對會超過3秒，因此要加大取得環境版本差異執行程式的執行時間，本次實驗專案執行時間為十分鐘。

另外，由於取得環境版本差異時，Lambda需要有兩個權限，第一個權限是羅列出該地區所有的Lambda Function，名稱為 lambda:ListFunctions ，第二個為取得該Function的權限，藉以查看該Function的Alias及其Configuration，名稱為 lambda:GetFunction ，該Policy最終會如下所示。

Tips:
當我們在配適合適的IAM權限給該執行程式的Role時，有時候會因為IAM系統太過於複雜，所以乾脆直接賦予Full Access權限，這樣在開發上雖然方便，但反而會因為權限開得太大而有不恰當的操作，開發人員或資安人員應當以配適最小的權限為原則進行配適，在這裡我提供一個方法，當不知道要搭配什麼樣的權限給當前的Role時，不彷乾脆讓程式噴Error Log，觀察Log中顯示的權限不足來進行權限配置，第二個做法是查詢[Boto 3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)，可以觀察所調用的Function Name來進行權限或資源上的配置。以本專案為例，調用的Function為[ListFunctions](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/lambda.html#Lambda.Client.list_functions)和[GetFunction](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/lambda.html#Lambda.Client.get_function)，因此直接賦予該Function所需的權限（名稱一樣）。

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "lambda:ListFunctions",
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "lambda:GetFunction",
                "logs:PutLogEvents"
            ],
            "Resource": [
                "arn:aws:lambda:*:*:function:*",
                "arn:aws:logs:us-east-1:xxxxxxxxxxxxxxx:log-group:/aws/lambda/stage-diff-list:*"
            ]
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "arn:aws:logs:us-east-1:xxxxxxxxxxx:*"
        }
    ]
}
```

取得環境差異名單相關設定及程式碼到這邊介紹完畢。

Reference: [Boto 3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
