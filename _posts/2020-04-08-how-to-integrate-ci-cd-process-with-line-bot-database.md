---
layout: post
title: 如何使用Line Bot整合CI/CD流程 - 資料庫
categories: [I.T.]
date: 2020-04-08 00:00:00
tags: [A.W.S,I.T.,CI/CD, Line Messaging API]
---
本篇是介紹資料庫在專案中扮演什麼樣的角色作為本篇介紹，關於本系列其他介紹請查看下列文章。

<!--more-->

Demo篇

[如何使用Line Bot整合CI/CD流程 - Admin Demo]({% post_url 2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin %})

[如何使用Line Bot整合CI/CD流程 - dev Demo]({% post_url 2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev %})

[如何使用Line Bot整合CI/CD流程 - Test Demo]({% post_url 2020-04-02-how-to-integrate-ci-cd-process-with-line-bot-demo-tester %})

開發篇

[如何使用Line Bot整合CI/CD流程 - 開發篇 Message API]({% post_url 2020-04-04-how-to-integrate-ci-cd-process-with-line-bot-dev-message-api %})

[如何使用Line Bot整合CI/CD流程 - 開發篇 取得環境差異名單]({% post_url 2020-04-05-how-to-integrate-ci-cd-process-with-line-bot-dev-stage-diff %})

[如何使用Line Bot整合CI/CD流程 - 開發篇 部署環境]({% post_url 2020-04-06-how-to-integrate-ci-cd-process-with-line-bot-dev-deploy-stage %})

Line篇

[如何使用Line Bot整合CI/CD流程 - Line篇 介面設定]({% post_url 2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface %})

<hr>

本專案中使用的資料庫為A.W.S.提供的MySQL 5.6.46，在本專案中，由於要達到部署、退版功能，會需要在參數中帶入相關資訊，假若只有少數的Function數量本可以在參數中直接帶入做傳送，但當要部署的功能數量為多數時，由於Line的參數數量字串長度限制，在傳遞時會造成Line API的Crash，因此我們需要要使用資料庫功能，做為要部署的中繼儲存體，當要進行部署、退版時，僅需帶入部署ID或命令給postBack事件，透過Lambda Function操作，便可達到部署、退版功能。本篇僅對資料庫的Schema對所需要的欄位進行本專案功能中運用介紹。

下面為Create Table的SQL Command

```sql
CREATE TABLE `DeployRecord` (
  `DeployID` int(11) NOT NULL,
  `DeployType` char(10) NOT NULL DEFAULT '',
  `FunctionName` char(255) NOT NULL DEFAULT '',
  `OriginVersion` char(50) NOT NULL DEFAULT '',
  `TargetVersion` char(50) DEFAULT NULL,
  `Deployer` char(255) NOT NULL DEFAULT '',
  `isDeploy` tinyint(1) NOT NULL DEFAULT '0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

1. DeployID 所扮演的角色為部署ID，本專案中乃是使用Python產生的時間戳記(TimeStamp)作為部署ID。
2. DeployType 所扮演的角色為要部署的類型，當使用者在Line機器然互動介面中進行查詢動作，可視為將進行部署動作，若使用者要進行將開發環境部署到測試環境，則會在該欄位中紀錄test，若要將測試環境部署到正式環境，則會在該欄位中紀錄prod。
3. FunctionName 為該次查詢到版本不同的Function Name，作為要部署的Function。
4. OriginVersion為要部署環境的版本，較特殊的是若為測試環境部署，該欄位會被記錄為$LATEST，表示開發環境的版本。
5. TargetVersion為被部署環境的版本，若進行退版，則可視為上一版的版本。
6. Deployer為Line使用者名稱，作為部署紀錄。
7. isDeploy為該查詢是否已經進行過部署，透過SQL語句，藉此能查詢最新部署資訊，若進行退版，當退版完成後會將被設置為False。

以上為資料庫作為本次專案中扮演儲存角色的介紹，較為簡易的原因是因為本專案僅作Demo中介紹使用，若需要可整合自家權限管理系統。
