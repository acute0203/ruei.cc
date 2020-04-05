---
layout: post
title: 如何使用Line Bot整合CI/CD流程 - Admin Demo
categories: [I.T.]
date: 2020-04-03 12:00:00
tags: [A.W.S,I.T.,CI/CD]
---

本篇是以Admin角色的Line功能視窗作為Demo，其他尚有Dev、Test角色請查閱其他文章。

<!--more-->

Demo篇

[如何使用Line Bot整合CI/CD流程 - dev Demo]({% post_url 2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev %})

[如何使用Line Bot整合CI/CD流程 - Test Demo]({% post_url 2020-04-02-how-to-integrate-ci-cd-process-with-line-bot-demo-tester %})

開發篇

[如何使用Line Bot整合CI/CD流程 - 開發篇 Message API]({% post_url 2020-04-04-how-to-integrate-ci-cd-process-with-line-bot-dev-message-api %})

[如何使用Line Bot整合CI/CD流程 - 開發篇 取得環境差異名單]({% post_url 2020-04-05-how-to-integrate-ci-cd-process-with-line-bot-dev-stage-diff %})

[如何使用Line Bot整合CI/CD流程 - 開發篇 部署環境]({% post_url 2020-04-06-how-to-integrate-ci-cd-process-with-line-bot-dev-deploy-stage %})

Line篇

[如何使用Line Bot整合CI/CD流程 - Line篇 介面設定]({% post_url 2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface %})

資料庫篇

[如何使用Line Bot整合CI/CD流程 - 資料庫]({% post_url 2020-04-08-how-to-integrate-ci-cd-process-with-line-bot-database %})

<hr>

點選環境佈署清單，可以點選目前要查看Dev->Test或Test->Production可行的佈署，以本文為例查看Dev->Test
![](/assets/2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin/1-1-Admin-env-deploy-list.jpg)

點擊後回傳目前Dev與Test所有版本不同的Function，並詢問是否要進行佈署
![](/assets/2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin/1-2-Admin-deploy-dev-confirm.jpg)

點擊Deploy進行佈署，開始時會回傳目前進度，結束後會回傳該佈署的ID
![](/assets/2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin/1-3-Admin-Deploy-Done.jpg)

點擊環境最新佈署人員，查看目前歷史佈署的人員及該部署的ID
![](/assets/2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin/1-4-Admin-Deploy-History.jpg)

點擊退版清單，查看環境最新可退的版本
![](/assets/2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin/1-5-Admin-RollBack-List.jpg)

點擊要退的環境版本，退版完成後回傳退版完成訊息
![](/assets/2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin/1-6-Admin-RollBack-Done.jpg)
