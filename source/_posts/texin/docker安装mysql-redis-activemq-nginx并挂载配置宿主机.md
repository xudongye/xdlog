---
title: centos7.4 部署 docker，nginx, mysql,redis,activemq 
date: 2020-09-10 14:21:07
categories: 
- 程序设计
tags:
- 微联电召
---


### mysql
```text
#安装docker
docker pull mysql

#先启动
docker run --name mysql -p 4148:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql

#注意：my.cnf得从docker mysql容器中复制一份到/texin/volume/mysql/conf中
docker cp mysql:/etc/mysql/my.cnf /texin/volume/mysql/conf

#成功后关闭mysql
#重新启动
#启动并挂载配置到宿主机
docker run -p 4148:3306 --name mysql \
 -v /texin/volume/mysql/log:/var/log/mysql \
 -v /texin/volume/mysql/data:/var/lib/mysql \
 -v /texin/volume/mysql/conf/my.cnf:/etc/mysql/my.cnf \
 -e MYSQL_ROOT_PASSWORD=root  \
 -d mysql
#进入mysql容器
docker exec -it mysql /bin/bash

#本地登录mysql
mysql -uroot -proot
#查询mysql用户
select user,host from mysql.user;
#新建用户@'%' 不限ip连接，只要本地则@'localhost'即可
create user 'weilian'@'%' identified by 'weilian2020';
#查看用户密码方式注意远程连接不支持：caching_sha2_password验证方式
#我们将他改成msyql_native_password
alter user 'weilian'@'%' identified with mysql_native_password BY 'weilian2020';
#刷新
flush privileges;
#查看用户密码方式，检查是否修改成功
select host,user,plugin,authentication_string from mysql.user;
#指定数据库texin授权用户
grant all privileges on texin.* to 'weilian'@'%';
#授予用户所有权限
grant all privileges on *.* to 'weilian'@'%';
#如果navicat还是连接失败请在挂载主机目录的my.cnf文件中加一句
default_authentication_plugin=mysql_native_password
#即可实现远程

```

### redis
````text
#安装redis默认lasted版本
docker pull redis
#启动并挂载
docker run -p 4149:6379 --name redis \
-v /texin/volume/redis/conf/redis.conf:/etc/redis/redis.conf \
-v /texin/volume/redis/data:/data \
-d redis redis-server --appendonly yes --requirepass 'weilian2020'
````

### mongo
```text
#安装mongo默认lasted版本
docker pull mongo
#启动并挂载
docker run \
--name mongo \
-p 4147:27017 \
-e MONGO_INITDB_ROOT_USERNAME='weilian' -e MONGO_INITDB_ROOT_PASSWORD='weilian2020' \
-v /texin/volume/mongo/data:/data/db \
-v /texin/volume/mongo/configdb:/data/configdb/ \
-d  mongo
```

### activemq
````text
#搜索activemq版本列表
docker search activemq
#安装activemq
docker pull webcenter/activemq
#启动指定对外端口61617和8167
docker run -d --name activemq -p 61617:61616 -p 8162:8161 webcenter/activemq

#挂载宿主机
#第一步：用临时方式启动一个activemq 容器
docker run --user root --privileged=true --rm -ti \
  -v  /texin/volume/activemq/conf:/mnt/conf \
  -v  /texin/volume/activemq/data:/mnt/data \
  webcenter/activemq /bin/sh
#第二步：将容器中的配置目录和数据目录拷贝出去并退出
chown activemq:activemq /mnt/conf
chown activemq:activemq /mnt/data
cp -a /opt/activemq/conf/* /mnt/conf/
cp -a /opt/activemq/data/* /mnt/data/
#第三步：运行activemq
docker run -d --name activemq -p 61617:61616 -p 8162:8161 \
-v /texin/volume/activemq/conf:/opt/activemq/conf \
-v /texin/volume/activemq/data:/opt/activemq/data \
-v /texin/volume/activemq/log:/var/log/activemq \
webcenter/activemq
````

### nginx
````text
#安装nginx
docker pull nginx
#启动
docker run -p 80:80 --name nginx \
-v /texin/volume/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /texin/volume/front:/etc/nginx/html \
-v /texin/volume/nginx/logs:/var/log/nginx  \
-d nginx

#在宿主机texin/volume/nginx/conf目录下新建nginx.conf
#内容(这是最终配置) 前后端分离资源访问
````

### nginx.conf

````text
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen      80;
        listen 443 ssl;
        server_name  dev.ruixuelong.com;

        ssl_certificate      cert/4670445_dev.ruixuelong.com.pem;
        ssl_certificate_key  cert/4670445_dev.ruixuelong.com.key;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers  on;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
         try_files $uri $uri/ @router;
          
        }

        location /payment/ {
            root html;
            try_files $uri $uri/ /payment/@router;
            #index index.html index.htm;
        }

	location /manage/ {
            root html;
            try_files $uri $uri/ /manage/@router;
            #index index.html index.htm;
        }


        location @router {
            rewrite ^.*$ /index.html last;
        }

        location /admin/ {
        set $cors '';
        if ($http_origin ~* 'https?://(localhost(:8090)?|dev\.ruixuelong\.com|ruixuelong\.com)') {
         set $cors 'true';
         }

        if ($cors = 'true') {
        add_header 'Access-Control-Allow-Origin' "$http_origin" always;
        add_header 'Access-Control-Allow-Credentials' 'true' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
        add_header 'Access-Control-Allow-Headers' 'Accept,Authorization,Cache-Control,Content-Type,DNT,If-Modified-Since,Keep-Alive,Origin,User-Agent,X-Mx-ReqToken,X-Requested-With' always;
        }

        if ($request_method = 'OPTIONS') {
        return 204;
        }

        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://47.94.90.143:8080/admin/;
        }

        location /taxi/ {
        set $cors '';
        if ($http_origin ~* 'https?://(localhost(:8090)?|dev\.ruixuelong\.com|ruixuelong\.com)') {
         set $cors 'true';
         }

        if ($cors = 'true') {
        add_header 'Access-Control-Allow-Origin' "$http_origin" always;
        add_header 'Access-Control-Allow-Credentials' 'true' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
        add_header 'Access-Control-Allow-Headers' 'Accept,Authorization,Cache-Control,Content-Type,DNT,If-Modified-Since,Keep-Alive,Origin,User-Agent,X-Mx-ReqToken,X-Requested-With' always;
        }

        if ($request_method = 'OPTIONS') {
        return 204;
        }

        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://47.94.90.143:8081/taxi/;
        }

        location /texin/ {
        set $cors '';
        if ($http_origin ~* 'https?://(localhost(:8090)?|dev\.ruixuelong\.com|ruixuelong\.com)') {
         set $cors 'true';
         }

        if ($cors = 'true') {
        add_header 'Access-Control-Allow-Origin' "$http_origin" always;
        add_header 'Access-Control-Allow-Credentials' 'true' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
        add_header 'Access-Control-Allow-Headers' 'Accept,Authorization,Cache-Control,Content-Type,DNT,If-Modified-Since,Keep-Alive,Origin,User-Agent,X-Mx-ReqToken,X-Requested-With' always;
        }

        if ($request_method = 'OPTIONS') {
        return 204;
        }

        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://47.94.90.143:8082/texin/;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
````

###注：所有端口记得要在aliyun安全规则添加规则并开放防火墙端口 