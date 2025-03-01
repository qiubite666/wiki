# 解析WordPress网站域名与常见问题

## 解析WordPress网站域名

通过公网IP地址直接访问您的WordPress网站会降低服务端的安全性。如果您已有域名或者想为WordPress网站注册一个域名，可以参考以下步骤。

1. 注册域名。

[阿里云域名注册流程](https://www.alibabacloud.com/help/zh/dws/user-guide/register-a-domain-name-on-alibaba-cloud#task-1830383)

[腾讯云域名注册流程](https://cloud.tencent.com/document/product/242/9595)

[华为云域名注册流程](https://support.huaweicloud.com/usermanual-domain/domain_ug_310004.html)

2. 备案域名。

   如果您的域名指向的网站托管在中国内地节点服务器，则需要到对应云平台上进行ICP备案，时长按照备案地处理速度，一般在1-30个工作日。如果不备案可能会导致域名无法访问。

3. 解析域名。将域名指向实例公网IP。

   域名解析是使用域名访问您的网站的必备环节。请在托管的云平台上进行解析，具体参考各云平台的文档，如需IPV6的访问请添加AAAA解析。



## 常见问题

### 1、WordPress中更新版本、上传主题或插件时，提示需要FTP登录凭证或无法创建目录

该问题可能是因为WordPress配置文件主题或插件的权限不足，您可以参考以下步骤解决。

1. 登录搭建WordPress的服务器

2. 运行以下命令，打开WordPress配置文件（wp-config.php文件位置请根据实际服务器上自定义的位置进行更换）

   ```shell
   sudo vim /usr/share/nginx/html/wordpress/wp-config.php
   ```

3. 按`i`键进入编辑模式。

4. 在最下方，添加如下代码。

```shell
define("FS_METHOD","direct");
define("FS_CHMOD_DIR", 0777);
define("FS_CHMOD_FILE", 0777);
```

5. 按`Esc`键，输入`:wq`后按`Enter`键，保存退出配置文件。

6. 返回WordPress仪表盘，刷新页面，可解决需要FTP登录凭证的问题。

   如果仍存在无法创建目录的问题，需再次返回登录，运行以下命令，将网站根目录的权限用户更新为Nginx对应的用户，本示例环境中为`www-data`用户。

```shell
sudo chown -R www-data /usr/share/nginx/html/wordpress
```



### 2、如果在设置过程中域名改错造成网站无法访问或需要将之前的公网IP换成新域名需要直接修改数据库

操作步骤如下：

1. 登录数据库，root密码具体看配置数据库时修改的数据。

   ```shell
   sudo mysql -u root -p
   ```


2. 查寻并修改数据库

   > 本示例将域名`www.example1.com`改为www.example2.com。

   ```shell
   MariaDB [(none)]> use wordpress;
   
   MariaDB [wordpress]> select * from wp_options where option_value like '%www.example1.com';
   
   +-----------+-------------+---------------------------+----------+
   | option_id | option_name | option_value              | autoload |
   +-----------+-------------+---------------------------+----------+
   |         2 | siteurl     | https://www.example1.com  | on       |
   |         3 | home        | https://www.example1.com  | on       |
   +-----------+-------------+---------------------------+----------+
   2 rows in set (0.046 sec)
   
   MariaDB [wordpress]> update wp_options set option_value=replace(option_value, 'https://www.example2.com', 'https://www.example2.com') where option_name = 'home' OR option_name = 'siteurl';
   
   MariaDB [wordpress]> exit;
   ```

   

3. 成功为WordPress网站设置新域名。



### 3、WordPress上传安装主题提示“您所关注的链接已过期”解决办法

在使用wordPress的时候，可能会遇到“您关注的链接已过期”或The link you followed has expired” 错误，一般从WordPress管理后台上传WordPress主题或插件到您的网站时，通常会发生此错误。WordPress有一个设置参数，可以控制从WordPress管理后台上传的文件的大小。



1. 登录服务器，查看配置文件位置

   ```shell
   ps -ef|grep php
   
   www-data 2754144 3454264  0 17:16 ?        00:00:01 php-fpm: pool www
   www-data 2754145 3454264  0 17:16 ?        00:00:01 php-fpm: pool www
   ops      2756021 2745748  0 17:22 pts/0    00:00:00 grep php
   root     3454264       1  0 Sep06 ?        00:00:40 php-fpm: master process (/etc/php/8.2/fpm/php-fpm.conf)
   
   可以确认配置文件在/etc/php/8.2/fpm/ 目录 修改该目录下的php.ini文件
   提前备份文件
   sudo cp /etc/php/8.2/fpm/php.ini /etc/php/8.2/fpm/php.ini.bak
   sudo vim /etc/php/8.2/fpm/php.ini
   
   打开文件后修改下面三行参数，保存并退出
   
   upload_max_filesize = 64M
   post_max_size = 64M
   max_execution_time = 300
   ```

   

2. 服务重新加载配置文件

   ```shell
   sudo systemctl reload php8.2-fpm.service
   ```

3. 修复完成，可以再次上传测试，限制调整为可以上传单个文件的最大大小为64M，最大执行时间为300秒。
