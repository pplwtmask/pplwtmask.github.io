---
title: 免费TLS证书使用
date: 2022-03-12 15:38:06
---

## 准备
1. VPS，一台服务器，幸运的话可以申请到oracle cloud终身免费的服务器。
2. 域名，免费的域名申请网站：[freenom](https://www.freenom.com/zh/index.html?lang=zh)，[自动续期](https://github.com/luolongfei/freenom)
3. 域名解析，[Cloudflare](https://www.cloudflare.com/zh-cn)提供免费的
4. 数据库，[db4free](https://db4free.net/)提供免费的，不要用于生产
5. ssh工具，国内外很多
6. 搭建一个网站，

## 安装 [acme.sh](https://github.com/acmesh-official/acme.sh)

### 执行脚本
```
wget -O -  https://get.acme.sh | sh
```

### 生效acme.sh
```
. .bashrc
```

### 开启自动升级
```
acme.sh --upgrade --auto-upgrade
```

### 生成证书
```
acme.sh --set-default-ca --server letsencrypt
# 以ECC证书为例
acme.sh --issue -d 二级域名.你的域名.com -w /home/vpsadmin/www/webpage --keylength ec-256 --force
```

### 安装证书
```
acme.sh --install-cert -d 二级域名.你的域名.com --ecc \
            --fullchain-file ~/nginx_cert/cert.pem \
            --key-file ~/nginx_cert/key.pem
```


### 证书定期更新
```
#!/bin/bash

/home/vpsadmin/.acme.sh/acme.sh --install-cert -d a-name.yourdomain.com --ecc --fullchain-file /home/vpsadmin/nginx_cert/cert.pem --key-file /home/vpsadmin/nginx_cert/key.pem

chmod +r /home/vpsadmin/nginx_cert/key.pem

```
```
$crontab -e

# 1:00am, 1st day each month, run `cert-renew.sh`
0 1 1 * *   bash /home/vpsadmin/nginx_cert/cert-renew.sh
```








