# **fail2ban**



## 简介

Fail2Ban 是一个 Linux 系统的应用软件，用来防止系统入侵，主要是防止各类型的暴力破解。它主要通过监控日志文件，一旦发现恶意攻击的登录请求，它会封锁对方的 IP 地址，使得对方无法再发起请求。



## 安装fail2ban

```shell
sudo apt install fail2ban
```



3. ### 配置需要监控封禁的服务





