---
layout: post
title: 如何使用Line Bot整合CI/CD流程 - 總覧
categories: [I.T.]
date: 2020-04-09 00:00:00
tags: [A.W.S,I.T.,CI/CD, Line Messaging API]
---
最近在為公司導入CI/CD流程，需要建構開發、測試、正式環境滿足業務需求，由於公司運行的架構採用的是A.W.S的Lambda Serverless環境運行，開發人員首先利用A.W.S Lambda開發環境進行線上開發，當開發人員開發完成，使用Postman透過A.W.S. API Gateway做測試，完成後便發行新的Lambda版本，使用Lambda的Alias Name分別劃分開發、測試、正式環境，並對版本產生連結。在這樣的情境下，當發行新Lambda版本或更新，若牽涉到的Lambda Function數過多，往往造成部署上的麻煩，因此開發Line Bot將佈署整合程序寫入Lambda，透過呼叫Lambda Function使用Line Bot做控制，針對開發過程中所涉及的角色功能做權限上的劃分。

<!--more-->

![如何使用Line Bot整合CI/CD流程 - 總覧圖](/assets/2020-04-09-how-to-integrate-ci-cd-process-with-line-bot-overview/CICD-Line-Overview-Diagram.png)

關於每個模組介紹，可參照上面編號逐一瀏覽。

1.Demo篇

* [如何使用Line Bot整合CI/CD流程 - Admin Demo]({% post_url 2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin %})

* [如何使用Line Bot整合CI/CD流程 - dev Demo]({% post_url 2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev %})

* [如何使用Line Bot整合CI/CD流程 - Test Demo]({% post_url 2020-04-02-how-to-integrate-ci-cd-process-with-line-bot-demo-tester %})

開發篇

2.[如何使用Line Bot整合CI/CD流程 - 開發篇 Message API]({% post_url 2020-04-04-how-to-integrate-ci-cd-process-with-line-bot-dev-message-api %})

3.[如何使用Line Bot整合CI/CD流程 - 開發篇 取得環境差異名單]({% post_url 2020-04-05-how-to-integrate-ci-cd-process-with-line-bot-dev-stage-diff %})

4.[如何使用Line Bot整合CI/CD流程 - 開發篇 部署環境]({% post_url 2020-04-06-how-to-integrate-ci-cd-process-with-line-bot-dev-deploy-stage %})

Line篇

5.[如何使用Line Bot整合CI/CD流程 - Line篇 介面設定]({% post_url 2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface %})

資料庫篇

6.[如何使用Line Bot整合CI/CD流程 - 資料庫]({% post_url 2020-04-08-how-to-integrate-ci-cd-process-with-line-bot-database %})
