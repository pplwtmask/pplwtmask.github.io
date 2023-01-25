---
title: Oracle Cloud配置
date: 2022-02-14 15:38:06
---



## 基本配置
Oracle Cloud的CentOS或Ubuntu自带系统，需要关闭防火墙和后台监管服务
> 注意关闭SELinux
> 1. 临时关闭 `# setenforce 0`
> 2. 永久关闭 /etc/selinux/config 'SELINUX=disabled' or `sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config;`

CentOS:
```
systemctl stop oracle-cloud-agent
systemctl disable oracle-cloud-agent
systemctl stop oracle-cloud-agent-updater
systemctl disable oracle-cloud-agent-updater
systemctl stop firewalld.service
systemctl disable firewalld.service
```

Ubuntu:
```
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT
iptables -F
apt-get purge netfilter-persistent -y
```

## 设置密码登录

```
#!/bin/bash
echo root:you password |sudo chpasswd
sudo sed -i 's/PermitRootLogin no/PermitRootLogin yes/g' /etc/ssh/sshd_config;
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config;
sudo systemctl restart sshd
```


