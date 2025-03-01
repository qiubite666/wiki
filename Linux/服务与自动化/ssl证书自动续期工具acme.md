# ssl证书自动续期工具——acme.sh

## 前言

新申请的网站想找一个https的免费证书，现在的免费证书有效期只有90天，于是想找到一个自动更新证书的工具acme.sh。由于我的域名解析在腾讯云上，我这里结合了腾讯云相关操作，来进行acme.sh服务部署。

## 下载acme.sh到你的 **home** 目录下

```shell
git clone https://github.com/acmesh-official/acme.sh.git

或直接到github进行下载 https://github.com/acmesh-official/acme.sh/releases
```



## 安装acme.sh

1. 创建一个**shell**的**alias**方便使用

   ```shell
   cd ~
   vim .bashrc
   alias acme.sh=~/.acme.sh/acme.sh
   
   保存退出后 重新加载环境变量
   source ~/.bashrc
   
   安装完成后会自动创建crontab任务检测证书有效期
   crontab -e 可看到定时任务
   13 5 * * * "/home/ops/.acme.sh"/acme.sh --cron --home "/home/ops/.acme.sh" > /dev/null
   ```

   

2. 注意换成自己的邮箱号

```shell
 acme.sh --install -m xxx@qq.com 
```

3. 设置自动升级

```shell
acme.sh --upgrade --auto-upgrade
```

4. 选择服务商

因为ZeroSSL支持泛域名并且兼容性更好，所以选择了ZeroSSL。

```shell
acme.sh --set-default-ca --server zerossl

关联一下自己的邮箱
acme.sh --register-account -m xxx@qq.com --server zerossl
```

如果需要Let's Encrypt服务商也可改为Let's Encrypt

```shell
acme.sh --set-default-ca --server letsencrypt

关联一下自己的邮箱
acme.sh --register-account -m xxx@qq.com --server letsencrypt
```



## 腾讯云创建api秘钥账户

![image-20250117003138813](./../images/image-20250117003138813.png)

1. 创建API秘钥

会得到SecretId和SecretKey

```shell
SecretId

AKIDVr1xxxxxxxxxxxxxxx

SecretKey

dgeU7Whixxxxxxxxxxxxxx
```



2. 将秘钥格式化导入环境变量

```shell
export Tencent_SecretId="xxxxx"
export Tencent_SecretKey="xxxxx"
```



3. `Tencent_SecretId` 和 `Tencent_SecretKey` 保存至 `~/.acme.sh/account.conf` 中，并在需要时自动获取，无需手动再设置。

## 申请证书

其中`*.aaa.com`是泛域名，`aaa.com`是根域名

```shell
acme.sh --dns dns_tencent --issue -d *.aaa.com -d aaa.com
```

命令执行完毕后，证书颁发成功

![image-20250117004249981](./../images/image-20250117004249981.png)



## 使用证书

### Nginx示例

Nginx 的配置项 `ssl_certificate` 需要使用 `/etc/nginx/ssl/fullchain.cer`

将证书文件复制到nginx配置的ssl证书目录

```shell
sudo cp ~/.acme.sh/\*.aaa.com_ecc/\*.aaa.com.cer /etc/nginx/cert/aaa.com.crt
sudo cp ~/.acme.sh/\*.aaa.com_ecc/\*.aaa.com.key /etc/nginx/cert/aaa.com.key
```



```shell
server {
        server_name aaa.com;
        server_name *.aaa.com;

        listen 80;
        listen [::]:80;
        listen 443 ssl;
        listen [::]:443 ssl;

        ssl_certificate /etc/nginx/cert/aaa.com.crt;
        ssl_certificate_key /etc/nginx/cert/aaa.com.key;

        location / {
                proxy_pass http://127.0.0.1;
        }
}
```

