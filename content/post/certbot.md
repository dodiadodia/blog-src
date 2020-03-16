---
title: "Certbot"
date: 2020-03-15T21:38:17+08:00
draft: false 
tags: ["cerbot", "ssl"]
isCJKLanguage: true
---

# 使用Certbot给网站申请通配符SSL证书

## 背景
最近在给组内多个k8s集群维护内部的ingress service，很多对外提供的https ingress服务都需要SSL证书，因此准备使用Let’s Encrypt的服务来申请SSL证书

## 介绍
2018年3月14日，Let’s Encrypt对外宣布ACME v2 已正式支持通配符证书。这就意外味着用户可以在 Let’s Encrypt 上免费申请支持通配符的 SSL 证书。

ACME v2 是 ACME 协议的更新版本，通配符证书只能通过 ACME v2 获得。要使用 ACME v2 协议申请通配符证书，只需一个支持该协议的客户端就可以了，官方推荐的客户端是 Certbot。

## Mac获取Certbot
Mac上获取Certbot很简单，直接brew就可以

`brew install certbot`

Linux平台获取Certbot

`wget https://dl.eff.org/certbot-auto`

## 申请证书

首先登录您的dns域名网站，准备好修改相关事宜

在申请Let’s Encrypt 证书的时候，需要校验域名的所有权，证明操作者有权利为该域名申请证书，目前支持三种验证方式：

- dns-01：给域名添加一个 DNS TXT 记录。
- http-01：在域名对应的 Web 服务器下放置一个 HTTP well-known URL 资源文件。
- tls-sni-01：在域名对应的 Web 服务器下放置一个 HTTPS well-known URL 资源文件。
> 通配符证书的申请，只能用上面的dns-01方式

例如我需要申请*.prod.thinkhcp.cloud的证书，需要使用下面的命令

`./certbot-auto certonly  -d "*.prod.thinkhcp.cloud" --manual --preferred-challenges dns-01  --server https://acme-v02.api.letsencrypt.org/directory`

> 参数说明<br>
> certonly表示插件<br>
> -d 表示需要申请的通配符域名<br>
> --perferred-challenges 表示使用dns-01方式申请，使用dns的txt校验方式验证<br>
> --server 需要使用的acme v2版本，需要显示指定

执行命令后，会需要输入邮箱和一些确认信息，然后根据提示的DNS TXT信息更新DNS服务的txt，等待20分钟左右，再确认即可

确认过程可以使用dig相关命令<br>
`dig -t txt _acme-challenge.prod.thinkhcp.cloud @8.8.8.8`
