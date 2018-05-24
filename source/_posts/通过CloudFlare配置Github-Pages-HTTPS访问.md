---
title: 通过CloudFlare配置Github Pages HTTPS访问
date: 2017-02-24 13:13:45
categories: website
tags: 
    - Github
    - Pages
    - HTTPS
    - SSL
    - CloudFlare
---

` 本文假设已经配置好GitHub Pages,并且已经配置好Custom Domain `
有关Github Pages的配置，请参考 [Github Pages](https://pages.github.com/)  

## DNS设置
### 将自定义域名的domain name servers 切换至CloudFlare.
![namserver](https://support.cloudflare.com/hc/en-us/article_attachments/205348367/find-namesevers.png)
```
elle.ns.cloudflare.com
rick.ns.cloudflare.com
```
自定义域名的相关DNS记录迁移到切换至CloudFlare.
一般情况下，几分钟后即能生效.
![dns](https://support.cloudflare.com/hc/en-us/article_attachments/205331408/status-active.png)
<!-- more -->
## SSL设置
### 进入CloudFlare控制台, 选择Crypto 菜单
将SSL模式切换至Full
![ssl](https://blog.cloudflare.com/content/images/2016/06/T08btVu.png)
此操作需要等待较长的时间，请耐心等待。
查看控制台状态，检查如果是Active Certificate，则是已经生效。

## Page Rule
### 设置Page Rule， 强制http访问转至HTTPS访问
![page](https://blog.cloudflare.com/content/images/2016/06/always_use_https_page_rule.png)

至此，通过CloudFlare 的SSL配置达到github pages的https访问支持。
例如: [xtimer.org](https://xtimer.org)
