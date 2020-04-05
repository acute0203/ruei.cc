---
layout: post
title: 如何使用Line Bot整合CI/CD流程 - Test Demo
categories: [I.T.]
date: 2020-04-02 12:00:00
tags: [A.W.S,I.T.,CI/CD]
---
最近在為公司導入CI/CD流程，需要建構開發、測試、正式環境滿足業務需求，由於公司運行的架構採用的是A.W.S的Lambda Serverless環境運行，開發人員首先利用A.W.S Lambda開發環境進行線上開發，當開發人員開發完成，使用Postman透過A.W.S. API Gateway做測試，完成後便發行新的Lambda版本，使用Lambda的Alias Name分別劃分開發、測試、正式環境，並對版本產生連結。在這樣的情境下，當發行新Lambda版本或更新，若牽涉到的Lambda Function數過多，往往造成部署上的麻煩，因此開發Line Bot將佈署整合程序寫入Lambda，透過呼叫Lambda Function使用Line Bot做控制，針對開發過程中所涉及的角色功能做權限上的劃分。

本篇是以Test角色的Line功能視窗作為Demo，其他尚有Admin、Dev角色請查閱其他文章。

<!--more-->

Demo篇

[如何使用Line Bot整合CI/CD流程 - dev Demo]({% post_url 2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev %})

[如何使用Line Bot整合CI/CD流程 - admin Demo]({% post_url 2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin %})

開發篇

[如何使用Line Bot整合CI/CD流程 - 開發篇 Message API]({% post_url 2020-04-04-how-to-integrate-ci-cd-process-with-line-bot-dev-message-api %})

[如何使用Line Bot整合CI/CD流程 - 開發篇 取得環境差異名單]({% post_url 2020-04-05-how-to-integrate-ci-cd-process-with-line-bot-dev-stage-diff %})

[如何使用Line Bot整合CI/CD流程 - 開發篇 部署環境]({% post_url 2020-04-06-how-to-integrate-ci-cd-process-with-line-bot-dev-deploy-stage %})

<hr>
點選測試/正式佈署清單，可以查看目前Test->Production可行的佈署，測試人員僅可將Function佈署到正式環境
![](/assets/2020-04-02-how-to-integrate-ci-cd-process-with-line-bot-demo-tester/3-1-test-Deploy-list.jpg)

點擊Deploy進行佈署，開始時會回傳目前進度，結束後會回傳該佈署的ID，點擊環境最新佈署人員，查看目前歷史佈署的人員及該部署的ID
![](/assets/2020-04-02-how-to-integrate-ci-cd-process-with-line-bot-demo-tester/3-2-tester-deploy-done-and-history.jpg)

點擊退版清單，查看環境最新可退的版本，測試人員可進行測試環境及正式環境的退版
![](/assets/2020-04-02-how-to-integrate-ci-cd-process-with-line-bot-demo-tester/3-3-tester-roll-back-list.jpg)

點擊要退的環境版本，退版完成後回傳退版完成訊息
![](/assets/2020-04-02-how-to-integrate-ci-cd-process-with-line-bot-demo-tester/3-4-tester-roll-back-done.jpg)
