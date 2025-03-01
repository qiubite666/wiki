# Wordpress的Nginx伪静态规则与常规安全设置

## Nginx伪静态规则

若在发布文章出现“此响应不是合法的JSON响应”，则需要检查nginx伪静态规则是否配置，具体配置如下

```shell
        location / {
                try_files $uri $uri/ /index.php?$args;
        }
```



如nginx无配置，在location /加入该配置即可。



