---
layout: post
title: 如何使用Line Bot整合CI/CD流程 - Dev Demo
categories: [I.T.]
date: 2020-04-03 12:00:00
tags: [A.W.S,I.T.,CI/CD]
---

本篇是以Dev角色的Line功能視窗作為Demo，其他尚有Admin、Test角色請查閱其他文章。

<!--more-->

DEMO篇

[如何使用Line Bot整合CI/CD流程 - Test Demo]({% post_url 2020-04-02-how-to-integrate-ci-cd-process-with-line-bot-demo-tester %})

[如何使用Line Bot整合CI/CD流程 - Admin Demo]({% post_url 2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin %})

開發篇

[如何使用Line Bot整合CI/CD流程 - 開發篇 Message API]({% post_url 2020-04-04-how-to-integrate-ci-cd-process-with-line-bot-dev-message-api %})

[如何使用Line Bot整合CI/CD流程 - 開發篇 取得環境差異名單]({% post_url 2020-04-05-how-to-integrate-ci-cd-process-with-line-bot-dev-stage-diff %})

[如何使用Line Bot整合CI/CD流程 - 開發篇 部署環境]({% post_url 2020-04-06-how-to-integrate-ci-cd-process-with-line-bot-dev-deploy-stage %})

Line篇

[如何使用Line Bot整合CI/CD流程 - Line篇 介面設定]({% post_url 2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface %})

資料庫篇

[如何使用Line Bot整合CI/CD流程 - 資料庫]({% post_url 2020-04-08-how-to-integrate-ci-cd-process-with-line-bot-database %})

<hr>

點選開發/測試佈署清單，可以查看目前Dev->Test可行的佈署，開發人員僅可將開發的Function佈署到測試環境
![](/assets/2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev/2-1-Dev-Deploy-List.jpg)

點擊Deploy進行佈署，開始時會回傳目前進度，結束後會回傳該佈署的ID
![](/assets/2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev/2-2-Dev-Deploy-Done.jpg)

點擊環境最新佈署人員，查看目前歷史佈署的人員及該部署的ID
![](/assets/2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev/2-3-Dev-Deploy-History.jpg)

點擊退版清單，查看環境最新可退的版本
![](/assets/2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev/2-4-Dev-RollBack-List.jpg)
