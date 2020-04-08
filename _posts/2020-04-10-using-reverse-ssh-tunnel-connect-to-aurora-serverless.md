---
layout: post
title: 使用SSH反向隧道連結AWS Aurora Serverless
categories: [I.T.]
date: 2020-04-10 00:00:00
tags: [A.W.S,I.T.]
---
使用AWS Aurora Serverless 服務作為後端資料庫時，成本節省，但該服務僅提供同一個位在AWS VPC內的裝置存取（[AWS官方文件](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless.html#aurora-serverless.limitations)），在這樣的條件下於Local端進行開發，往往造成極大的不方便，若使用Aurora Serverless服務提供的Endpoint存取資料庫時，呈現的會是連不上的情形。如下圖所示

<!--more-->

![](/assets/2020-04-10-using-reverse-ssh-tunnel-connect-to-aurora-serverless/1-ide-connect-fail.png)

為解決上述Local端開發問題，本篇文章介紹，讓地端的資料庫IDE（如Workbench）連結上Aurora Serverless做Debug，使用SSH Revere Tunnel連結，關於SSH Revere Tunnel介紹，將在後續提出。

P.S.在開始之前，您必須先確認您的AWS帳號是否有開設EC2的權限。

開設EC2，進入到AWS EC2 Console，點選Launch Instance，進入到選擇AMI介面，為快速設定及節省成本，可選擇Free tier 的 Amazon Linux AMI。

![](/assets/2020-04-10-using-reverse-ssh-tunnel-connect-to-aurora-serverless/2-create-ec2-ami.png)

因為僅供連結使用，機器規格並不需要設定到太高階，為節省成本，使用Free tier的t2.micro，接著點選Next: Configure Instance Details。

![](/assets/2020-04-10-using-reverse-ssh-tunnel-connect-to-aurora-serverless/3-instance-type.png)

因為[使用限制](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless.html#aurora-serverless.limitations)，Network選項「必須」選擇和Aurora Serverless同一個VPC，其他選項皆可保持預設。

![](/assets/2020-04-10-using-reverse-ssh-tunnel-connect-to-aurora-serverless/4-vpc-setting.png)

Add Storage可視其他需求而定，本範例中僅開設最小的8GB。

![](/assets/2020-04-10-using-reverse-ssh-tunnel-connect-to-aurora-serverless/5-storage.png)

關於SecurityGroup設定，需打開22 port供Reverse SSH Tunnel連入，並記錄該EC2 SecurityGroup ID。

![](/assets/2020-04-10-using-reverse-ssh-tunnel-connect-to-aurora-serverless/6-ec2-security-group.png)

最後點選Launch，選擇您適合的SSH KEY供待會SSH Tunnel連入時使用，最後點選Launch Instances。

![](/assets/2020-04-10-using-reverse-ssh-tunnel-connect-to-aurora-serverless/7-ssh-key-console.png)

當EC2 Instance狀態從Initializing轉換成2/2 checks，即可連入。

記錄您的EC2 Public DNS (IPv4)，若供未來長期使用，建議使用Elastic IP，當EC2關閉時時才不需要對連結的Endpoint做修改。

![](/assets/2020-04-10-using-reverse-ssh-tunnel-connect-to-aurora-serverless/8-ec2-public-ip.png)

接下來開啟您的AWS RDS Console，點選您的Aurora Serverless DB identifier，在Connectivity & security頁籤底下記錄您的Endpoint，待會在設定SSH Reverse Tunnel會用到，接著點選RDS的VPC security groups。

![](/assets/2020-04-10-using-reverse-ssh-tunnel-connect-to-aurora-serverless/9-rds-endpoint.png)

在Inbound rules底下，為該VPC security groups新增MYSQL/Aurora選項，並將EC2的Security Group ID帶入。

![](/assets/2020-04-10-using-reverse-ssh-tunnel-connect-to-aurora-serverless/10-rds-security-group.png)

接下來要在Local端設定SSH Reverse Tunnel，以Mac為例，首先打開您的TERMINAL視窗，輸入

```script
vi ~/.ssh/config
```

i進入編輯模式，在底下加入

```
# Serverless Tunnel
Host {輸入您的名稱，如dev}
HostName {剛剛紀錄EC2 Public DNS (IPv4)}
User {使用者名稱，如當時開設使用的AMI為Amazon Linux AMI，則為ec2-user}
IdentitiesOnly yes
IdentityFile {開設EC2使用的SSH Key存放位置}
LocalForward 3306 {Aurora Serverless的Endpoint}:3306
```

編輯完成後儲存離開介面

接下來在Terminal底下輸入

```script
ssh {您剛剛在HOST輸入的名稱} -Nv
```

![](/assets/2020-04-10-using-reverse-ssh-tunnel-connect-to-aurora-serverless/11-ssh-connect-1.png)

![](/assets/2020-04-10-using-reverse-ssh-tunnel-connect-to-aurora-serverless/12-ssh-connect-2.png)


回到資料庫IDE（如Workbench）視窗，Host輸入 127.0.0.1，輸入您存取資料庫的帳號及密碼，並且連線。

![](/assets/2020-04-10-using-reverse-ssh-tunnel-connect-to-aurora-serverless/13-ide-prepare.png)

便可連入RDS Aurora Serverless Database

![](/assets/2020-04-10-using-reverse-ssh-tunnel-connect-to-aurora-serverless/14-ide-connect-success.png)
