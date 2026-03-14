---
title: 通过 Docker 在 VPS 上部署 Sub-Store
published: 2026-01-01
description: '使用 Docker 快速部署 Sub-Store，实现订阅聚合与管理。'
image: '../../assets/images/Sub-Store.png'
tags: [教程, 工具, VPS]
category: 工具
draft: false
---
## 前言
Sub-Store 是目前最强大的高级订阅管理工具。不同于基础的订阅转换，它更像是一个针对代理配置的“后端工作站”。

我个人主要将其用于以下场景：

* 多订阅聚合管理：将多个机场订阅和零散的自建节点整合，去重并重命名，保持节点列表整洁。

* 配置动态生成：利用其强大的文件存取和脚本（Script）功能，动态生成适配 Sing-box 的 JSON 配置。

* 一链接走天下：provider 字段中填入订阅，实现只需一条 Sub-Store 链接，即可在 iOS、Android、Windows 等全平台（Mihomo/Sing-box）实现代理通用且自动更新。
::github{repo="sub-store-org/Sub-Store"}    
[官方频道](https://t.me/zhetengsha)  
[官方群组](https://t.me/zhetengsha_group)

## 🛠️ 环境准备
1C 512M即可 使用机器为debian13  
首先，安装必要的依赖工具

```bash
apt update && apt install -y curl sudo
```

## 🛠️ 安装 Docker 
:::tip[提示]
如果你已经安装过 Docker，可以跳过此步骤。
:::

使用官方脚本一键安装docker

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

## 🛠️ 安装Substore
:::tip[警告]
SUB_STORE_FRONTEND_BACKEND_PATH为substore api，是确保安全的唯一凭证，泄露后他人可随意访问你存的订阅  
建议至少20位  *本人曾有案例刚部署完50分钟内被扫出来，订阅全部泄露*  
* **请务必更改!**
* **请务必更改!!** 
* **请务必更改!!!**  
推荐于[1password](https://1password.com/zh-cn/password-generator)生成*
:::
:::tip[提示]
可选docker run或docker compose形式
:::
#### Docker run
```yaml
docker run -d \
    --restart=always \
    -p 127.0.0.1:3001:3001 \    #仅暴露内网，需反代使用
    -e "TZ=Asia/Shanghai" \    #设置上海时区，方便观察日志
    -e "SUB_STORE_BACKEND_SYNC_CRON=55 23 * * *" \    #每日23点55分自动拉取订阅更新
    -e "SUB_STORE_FRONTEND_BACKEND_PATH=/63PC2UpfxJicqFu84Zk7" \   #务必更改!!!设置api，此为确保安全的唯一凭证，切勿泄露，可使用随机密码生成，越长则越不容易被扫出来
    -e "SUB_STORE_BODY_JSON_LIMIT=20mb" \    #设置备份还原最大文件大小为20mb，防止备份文件过大还原报错
    -v /root/sub-store:/opt/app/data \
    --name sub-store \
    xream/sub-store:latest
```
### Docker Compose  
自行保存为docker-compose.yml后于对应目录运行

```yaml
services:
  sub-store:
    image: xream/sub-store:latest
    container_name: sub-store
    restart: always
    ports:
      - "127.0.0.1:3001:3001"
    environment:
      TZ: "Asia/Shanghai"
      SUB_STORE_BACKEND_SYNC_CRON: "55 23 * * *"
      SUB_STORE_FRONTEND_BACKEND_PATH: "/63PC2UpfxJicqFu84Zk7"    #必须更改!!!设置api，此为确保安全的唯一凭证
      SUB_STORE_BODY_JSON_LIMIT: "20mb"
    volumes:
      - "/root/sub-store:/opt/app/data"
```
```bash
docker compose up -d
```
## 🛠️ 使用caddy反代
:::tip[提示]
已安装可跳过  
请自行解析域名至你的服务器,也可使用宝塔(nginx)等反代
:::
官方方式安装caddy
```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```
打开/etc/caddy/Caddyfile，添加以下字段覆盖原文
```caddy
substore.xxx.com {   #你的域名
    reverse_proxy localhost:3001
}
```
重载caddy并查看日志
```bash
systemctl reload caddy
```
```bash
journalctl -u caddy -f
```
如无报错等问题，则输入如下网址访问你的substore界面
```bash
https://你的域名/subs?api=https://你的域名/你设置的api
```