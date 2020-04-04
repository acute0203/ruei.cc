---
layout: post
title: 如何使用Line Bot整合CI/CD流程 - Dev Demo
categories: [I.T.]
date: 2020-04-03 12:00:00
tags: [A.W.S,I.T.,CI/CD]
---
最近在為公司導入CI/CD流程，需要建構開發、測試、正式環境滿足業務需求，由於公司運行的架構採用的是A.W.S的Lambda Serverless環境運行，開發人員首先利用A.W.S Lambda開發環境進行線上開發，當開發人員開發完成，使用Postman透過A.W.S. API Gateway做測試，完成後便發行新的Lambda版本，使用Lambda的Alias Name分別劃分開發、測試、正式環境，並對版本產生連結。在這樣的情境下，當發行新Lambda版本或更新，若牽涉到的Lambda Function數過多，往往造成部署上的麻煩，因此開發Line Bot將佈署整合程序寫入Lambda，透過呼叫Lambda Function使用Line Bot做控制，針對開發過程中所涉及的角色功能做權限上的劃分。

本篇是以Dev角色的Line功能視窗作為Demo，其他尚有Admin、Test角色請查閱其他文章。

<!--more-->

DEMO篇

[如何使用Line Bot整合CI/CD流程 - Test Demo]({% post_url 2020-04-02-how-to-integrate-ci-cd-process-with-line-bot-demo-tester %})

[如何使用Line Bot整合CI/CD流程 - Admin Demo]({% post_url 2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin %})

開發篇

[如何使用Line Bot整合CI/CD流程 - 開發篇 Message API]({% post_url 2020-04-04-how-to-integrate-ci-cd-process-with-line-bot-dev-lambda %})

<hr>

點選開發/測試佈署清單，可以查看目前Dev->Test可行的佈署，開發人員僅可將開發的Function佈署到測試環境
![](/assets/2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev/2-1-Dev-Deploy-List.jpg)

點擊Deploy進行佈署，開始時會回傳目前進度，結束後會回傳該佈署的ID
![](/assets/2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev/2-2-Dev-Deploy-Done.jpg)

點擊環境最新佈署人員，查看目前歷史佈署的人員及該部署的ID
![](/assets/2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev/2-3-Dev-Deploy-History.jpg)

點擊退版清單，查看環境最新可退的版本
![](/assets/2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev/2-4-Dev-RollBack-List.jpg)
