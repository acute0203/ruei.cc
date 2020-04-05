---
layout: post
title: 如何使用Line Bot整合CI/CD流程 - Line篇 介面設定
categories: [I.T.]
date: 2020-04-07 00:00:00
tags: [A.W.S,I.T.,CI/CD, Line Messaging API]
---
最近在為公司導入CI/CD流程，需要建構開發、測試、正式環境滿足業務需求，由於公司運行的架構採用的是A.W.S的Lambda Serverless環境運行，開發人員首先利用A.W.S Lambda開發環境進行線上開發，當開發人員開發完成，使用Postman透過A.W.S. API Gateway做測試，完成後便發行新的Lambda版本，使用Lambda的Alias Name分別劃分開發、測試、正式環境，並對版本產生連結。在這樣的情境下，當發行新Lambda版本或更新，若牽涉到的Lambda Function數過多，往往造成部署上的麻煩，因此開發Line Bot將佈署整合程序寫入Lambda，透過呼叫Lambda Function使用Line Bot做控制，針對開發過程中所涉及的角色功能做權限上的劃分。

<!--more-->

本篇是介紹Line的Rich Menu作為本篇介紹，關於本系列其他介紹請查看下列文章。

Demo篇

[如何使用Line Bot整合CI/CD流程 - Admin Demo]({% post_url 2020-04-03-how-to-integrate-ci-cd-process-with-line-bot-demo-admin %})

[如何使用Line Bot整合CI/CD流程 - dev Demo]({% post_url 2020-04-01-how-to-integrate-ci-cd-process-with-line-bot-demo-dev %})

[如何使用Line Bot整合CI/CD流程 - Test Demo]({% post_url 2020-04-02-how-to-integrate-ci-cd-process-with-line-bot-demo-tester %})

開發篇

[如何使用Line Bot整合CI/CD流程 - 開發篇 Message API]({% post_url 2020-04-04-how-to-integrate-ci-cd-process-with-line-bot-dev-message-api %})

[如何使用Line Bot整合CI/CD流程 - 開發篇 取得環境差異名單]({% post_url 2020-04-05-how-to-integrate-ci-cd-process-with-line-bot-dev-stage-diff %})

[如何使用Line Bot整合CI/CD流程 - 開發篇 部署環境]({% post_url 2020-04-06-how-to-integrate-ci-cd-process-with-line-bot-dev-deploy-stage %})


<hr>

本篇主要介紹Line Rich Menu相關的設定，Line Rich Menu是作為本專案中使用者的溝通介面，當使用者點選了Line機器人，當中下方選單即為基礎的溝通介面，溝通介面有兩種設計方式，一種為使用[Line Official Account manager](https://developers.line.biz/en/)設計上傳，另一種為透過Line Developer撰寫Json File設定，前者較為容易，只需在該頁面上針對所需要的功能及文字做設定，便可達到所需要的功能，較適合初學者使用，但若設計的版面較為複雜客製化或需要對使用者顯示相對應的頁面，則需要使用Line Developer撰寫Json File設計，本篇以介紹後者為主。

首先要設計Rich Menu，可以參考[Line Rich Menu](https://developers.line.biz/en/reference/messaging-api/#create-rich-menu)，該文件為Line官方的文件，本篇亦是參考該文件撰寫，透過PostMan傳遞相關參數達成創建目的。

預先準備工作：
1. Channel access token
所有的Rich Menu操作，都需要使用該機器人的Channel Access Token，該Access Token可於[Line Official Account manager](https://developers.line.biz/en/)中取得
登入後即可到達下列頁面
![](/assets/2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface/1-Line-Developer-Login.png)
點選「Providers」頁籤，即可看到您目前管理的所有Line機器人，並且點選您專案要使用的機器人
![](/assets/2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface/2-Line-Developer-provider.png)
進入後即可看到您Line機器人的基本設定
![](/assets/2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface/3-Line-Developer-Basic.png)
點選Messaging API頁籤，並向下滑動，到Webhook settings選項，可以設定當機器人收到訊息時，要傳送到哪個API網址做處理
![](/assets/2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface/4-Line-Developer-webhook.png)
再繼續向下滑動到最底端，即可看到您的Channel access token，我們將全程使用這個Channel access token來和Line的機器人API做溝通。
![](/assets/2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface/5-Line-Developer-Channel-access-token.png)

2. Background Image
若要開發Rich Menu，所有的Rich Menu都必須要上傳一張圖片做為該Rich Menu的背景圖，本專案只簡單產生單色背景圖，由於要設計Admin、Developer、Tester三種介面，每種介面僅需要三種不同功能，因此簡單畫上線條當做按鈕邊界的分割，作為底圖，並依序為三種角色寫上該按鈕所代表的功能。

底圖
![](/assets/2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface/6-simple-bg.png)

Admin介面
![](/assets/2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface/7-simple-bg-admin.png)

Developer介面
![](/assets/2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface/8-simple-bg-dev.png)

Tester介面
![](/assets/2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface/9-simple-bg-tester.jpeg)

這三張圖的大小皆為2500 X 843 ，大小數字將會在下面就紹中使用到。

前置工作準備完畢！接下來便開始進入本文主題，創立介面！如前文中所提，所有的介面創立皆可使用Postman傳送Json檔達成，但需要在所有的API Header加入

Authorization:Bearer {Channel access token}
Content-Type:application/json

首先第一步為創立Rich menu
以Admin的Rich Menu為例，Json如下：
```json
{
    "size": {
      "width": 2500,
      "height": 843
    },
    "selected": true,
    "name": "Admin RichMenu",
    "chatBarText": "Admin CI/CD",
    "areas": [
      {
        "bounds": {
          "x": 0,
          "y": 0,
          "width": 833,
          "height": 843
        },
        "action": {
			"type":"message",
			"label":"環境最新\n佈署人員",
			"text":"環境最新佈署人員"
		}
      },
      {
        "bounds": {
          "x": 834,
          "y": 0,
          "width": 833,
          "height": 843
        },
        "action": {
			"type":"message",
			"label":"退版清單",
			"text":"退版清單"
		}
      },
      {
        "bounds": {
          "x": 1667,
          "y": 0,
          "width": 834,
          "height": 843
        },
        "action": {
			"type":"message",
			"label":"環境佈署清單",
			"text":"環境佈署清單"
		}
      }
   ]
}
```
值得注意的是第一層JSON中，Key為Size的Width和height的數值，即為我們剛剛在準備步驟中所準備的背景圖大小，這大小可因設計需求不同而調整。
第二個是Key為area中，bounds所代表的意義，X,Y數值為該按鈕功能位置的起點，Width和height為該功能的邊界，藉此框出該功能，當點擊範圍座落在該功能邊界內，即表示點擊該功能，如下所示

![](/assets/2020-04-07-how-to-integrate-ci-cd-process-with-line-bot-line-interface/10-simple-bg-admin-detail.png)

設計JSON檔完成後，便可使用POST方法，傳送至

https://api.line.me/v2/bot/richmenu

P.S. Header記得要帶入

回傳為

```json
{
  "richMenuId": "{richMenuId}"
}
```

記錄此ID，方便後續操作。

接下來要為Rich Menu上傳背景圖，有趣的是藉由API創建的Rich Menu，都必須要有背景圖才可以使用。

Postman亦可上傳圖片，只要在body中點選Binary，便可選擇要上傳的檔案，待選擇完畢，便可使用POST方法，傳送至

https://api-data.line.me/v2/bot/richmenu/{richMenuId}/content

其中{richMenuId}請替換為上一步驟中系統回傳的richMenuId，Header中的Content-Type請替換為 image/jpeg 或 image/png

成功回傳為一空的Json Object

```json
{}
```

接下來便可為CI/CD中各種角色創立各種介面，完成後可藉由GET方法，呼叫

https://api.line.me/v2/bot/richmenu/list

P.S. Header記得要帶入

便可取得剛剛創立所有的介面資料，如

```json
{
    "richmenus": [
        {
            "richMenuId": "richmenu-cccccccccccccccccccccccccccf",
            "name": "Tester RichMenu",
            "size": {
                "width": 2500,
                "height": 843
            },
            "chatBarText": "Tester CI/CD",
            "selected": true,
            "areas": [
                {
                    "bounds": {
                        "x": 0,
                        "y": 0,
                        "width": 833,
                        "height": 843
                    },
                    "action": {
                        "label": "環境最新\n佈署人員",
                        "type": "message",
                        "text": "環境最新佈署人員"
                    }
                },
                {
                    "bounds": {
                        "x": 834,
                        "y": 0,
                        "width": 833,
                        "height": 843
                    },
                    "action": {
                        "label": "退版清單",
                        "type": "message",
                        "text": "退版清單"
                    }
                },
                {
                    "bounds": {
                        "x": 1667,
                        "y": 0,
                        "width": 834,
                        "height": 843
                    },
                    "action": {
                        "label": "測試/正式環境",
                        "type": "postback",
                        "data": "deployEnv=prod"
                    }
                }
            ]
        },
        {
            "richMenuId": "richmenu-bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
            "name": "Admin RichMenu",
            "size": {
                "width": 2500,
                "height": 843
            },
            "chatBarText": "Admin CI/CD",
            "selected": true,
            "areas": [
                {
                    "bounds": {
                        "x": 0,
                        "y": 0,
                        "width": 833,
                        "height": 843
                    },
                    "action": {
                        "label": "環境最新\n佈署人員",
                        "type": "message",
                        "text": "環境最新佈署人員"
                    }
                },
                {
                    "bounds": {
                        "x": 834,
                        "y": 0,
                        "width": 833,
                        "height": 843
                    },
                    "action": {
                        "label": "退版清單",
                        "type": "message",
                        "text": "退版清單"
                    }
                },
                {
                    "bounds": {
                        "x": 1667,
                        "y": 0,
                        "width": 834,
                        "height": 843
                    },
                    "action": {
                        "label": "環境佈署清單",
                        "type": "message",
                        "text": "環境佈署清單"
                    }
                }
            ]
        },
        {
            "richMenuId": "richmenu-aaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
            "name": "Dev RichMenu",
            "size": {
                "width": 2500,
                "height": 843
            },
            "chatBarText": "Dev CI/CD",
            "selected": true,
            "areas": [
                {
                    "bounds": {
                        "x": 0,
                        "y": 0,
                        "width": 833,
                        "height": 843
                    },
                    "action": {
                        "label": "環境最新\n佈署人員",
                        "type": "message",
                        "text": "環境最新佈署人員"
                    }
                },
                {
                    "bounds": {
                        "x": 834,
                        "y": 0,
                        "width": 833,
                        "height": 843
                    },
                    "action": {
                        "label": "退版清單",
                        "type": "message",
                        "text": "退版清單"
                    }
                },
                {
                    "bounds": {
                        "x": 1667,
                        "y": 0,
                        "width": 834,
                        "height": 843
                    },
                    "action": {
                        "label": "開發/測試環境",
                        "type": "postback",
                        "data": "deployEnv=test"
                    }
                }
            ]
        }
    ]
}
```

關於Rich Menu的Object，可以參考官方文件[Rich Menu Object](https://developers.line.biz/en/reference/messaging-api/#rich-menu-object)

最後我們可以依照每個角色賦予合適的介面及權限，使用POST方法，呼叫

https://api.line.me/v2/bot/user/{userId}/richmenu/{richMenuId}

P.S. Header記得要帶入

userId帶入該角色的LineID，richMenuId帶入該角色合適的RichMenu即可。

若要取消該角色與RichMenu連結，則使用DELETE方法，呼叫

https://api.line.me/v2/bot/user/{userId}/richmen

P.S. Header記得要帶入

便可取消

其他相關設定請參閱[文件](https://developers.line.biz/en/reference/messaging-api/)

以上，Line介面設定完成。

Reference: [messaging-api Documentation](https://developers.line.biz/en/reference/messaging-api/)
