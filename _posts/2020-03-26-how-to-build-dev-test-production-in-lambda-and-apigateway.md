---
layout: post
title: 在AWS Lambda環境下開發如何設定Dev、Test、Production三種環境的API Gateway
categories: [I.T.]
date: 2020-03-26 12:00:00
tags: [A.W.S,I.T.]
---
最近在為公司導入CI/CD流程，需要建構開發、測試、正式環境滿足業務需求，由於公司的後端開發環境全部都基於AWS的Serverless架構下開發，使用到了API Gateway、Lambda，因此紀錄一下如何在這樣的架構下將環境分別切開。

<!--more-->
<hr>
先到Lambda創建新版本
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/1-Lambda-create-new-version.png)

Version description部分可以留空，或針對版本變化描述
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/2-Lambda-create-new-version-setting.png)

以測試環境做為演示，創建新的別名
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/3-Lambda-create-new-alias.png)

test連結到新創版號
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/4-Lambda-create-new-alias-setting.png)

創建結果
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/5-Lambda-alias setting-result.png)

進到該LambdaAPI Gateway，並創建test stage
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/7-APIGateway-create-new-stage.png)

test stage Variable下創建key為lambdaAlias,value為 test
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/8-APIGateway-stageVariable-setting.png)

回到Resource頁，需針對每個呼叫方法設定，點選Integration Request
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/6-APIGateway-page.png)

![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/9-APIGateway-IR-Page.png)

修改Lambda Function名稱為function Name:${stageVariables.lambdaAlias}
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/10-APIGateway-IR-Page-setting.png)

針對提醒新增call function的權限
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/11-APIGateway-IR-Page-setting-alert.png)

以本次實作為例，跑一次cmd，修改字串command中${stageVariables.lambdaAlias}為test
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/12-APIGateway-run-cmd.png)

在Resource中Actions裡deploy環境
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/13-APIGateway-resource-actions-deploy.png)

點選test
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/14-APIGateway-deploy-stage-setting.png)

切換到custom domain頁，選擇要修改的Custom domain names
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/15-APIGateway-Custom-domain-page.png)

13.點選Configure API mappings
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/16-APIGateway-Configure-API-Mapping.png)

點選要新增或修改API的Stage
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/17-APIGateway-add-API-Mapping.png)

新增
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/18-APIGateway-add-API-Mapping-setting.png)

使用post man送出請求
送出測試環境Request
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/19-postman-test.png)

送出開發環境Request
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/20-postman-dev.png)

查看CloudWatch Lambda是否有針對不同的環境產生不同的log
![](/assets/2020-03-26-how-to-build-dev-test-production-in-lambda-and-apigateway/21-cloudwatch-diff.png)

完成！
