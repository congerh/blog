---
title: "Install Zabbix 3.4 on Debian 9 with Nginx"
categories:
  - Zabbix
tags:
  - Zabbix
  - Installation Guide
---

Install Zabbix 3.4 server/proxy, agent on Debian 9 with nginx and MySQL.

## Adding Zabbix repository

```shell
wget https://repo.zabbix.com/zabbix/3.4/debian/pool/main/z/zabbix-release/zabbix-release_3.4-1+stretch_all.deb
dpkg -i zabbix-release_3.4-1+stretch_all.deb
apt update
```

## Install Zabbix server, frontend, agent

```shell
apt install zabbix-agent zabbix-server-mysql zabbix-frontend-php
# or
apt install zabbix-agent  zabbix-proxy-mysql
apt-get install php7.0 php7.0-fpm php7.0-mysql php7.0-bcmath php7.0-gd php7.0-ldap php7.0-mbstring php7.0-xml
```

## Create initial MySQL database

```shell
mysql -uroot -p
password
# zabbix-server-mysql
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> grant all privileges on zabbix.* to zabbix@localhost identified by 'password';
mysql> quit;
# or zabbix-proxy-mysql
mysql> create database zabbix_proxy character set utf8 collate utf8_bin;
mysql> grant all privileges on zabbix_proxy.* to zabbix@localhost identified by 'password';
mysql> flush privileges;
```

## Importing data

```shell
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
# or
shell> cd database/mysql
shell> mysql -uzabbix -p<password> zabbix < schema.sql
# zabbix_proxy
shell> mysql -uzabbix -p<password> zabbix_proxy < schema.sql
# stop here if you are creating database for Zabbix proxy
shell> mysql -uzabbix -p<password> zabbix < images.sql
shell> mysql -uzabbix -p<password> zabbix < data.sql
```

## Configure the database for Zabbix server

Edit file /etc/zabbix/zabbix_server.conf

```shell
vi /etc/zabbix/zabbix_server.conf
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=<password>
```

## Configure Zabbix Proxy

```shell
vi /etc/zabbix/zabbix_proxy.conf
DBHost=localhost
DBName=zabbix_proxy
DBUser=zabbix
DBPassword=<password>


```

## Configure PHP for Zabbix frontend

Edit file php.ini <https://github.com/zabbix/zabbix-docker/blob/3.4/web-nginx-mysql/ubuntu/conf/etc/php/7.2/fpm/conf.d/99-zabbix.ini>

```shell
max_execution_time=300
memory_limit=128M
post_max_size=16M
upload_max_filesize=2M
max_input_time=300
always_populate_raw_post_date=-1
# max_input_vars=10000
# date.timezone=Asia/Shanghai
# session.save_path=/var/lib/php7
```

## Copy Zabbix frontend PHP files

```shell
/usr/share/zabbix
vi /usr/share/zabbix/include/defines.inc.php # change FONT_NAME DejaVuSans to other font such as msyh for supporting chinese language

```

## Configure NGINX

Edit nginx.conf <https://github.com/zabbix/zabbix-docker/blob/3.4/web-nginx-mysql/ubuntu/conf/etc/nginx/nginx.conf>

```shell
user www-data;
worker_processes 5;
#worker_rlimit_nofile 256000;

error_log /dev/fd/2 warn;

pid        /var/run/nginx.pid;

events {
    worker_connections 5120;
    use epoll;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /dev/fd/1 main;

    client_body_timeout             5m;
    send_timeout                    5m;

    connection_pool_size            4096;
    client_header_buffer_size       4k;
    large_client_header_buffers     4 4k;
    request_pool_size               4k;
    reset_timedout_connection       on;


    gzip                            on;
    gzip_min_length                 100;
    gzip_buffers                    4 8k;
    gzip_comp_level                 5;
    gzip_types                      text/plain;
    gzip_types                      application/x-javascript;
    gzip_types                      text/css;

    output_buffers                  128 512k;
    postpone_output                 1460;
    aio                             on;
    directio                        512;

    sendfile                        on;
    client_max_body_size            8m;
    client_body_buffer_size         256k;
    fastcgi_intercept_errors        on;

    tcp_nopush                      on;
    tcp_nodelay                     on;

    keepalive_timeout               75 20;

    ignore_invalid_headers          on;

    index                           index.php;
    server_tokens                   off;

    include /etc/nginx/conf.d/*.conf;
}

```

zabbix.conf <https://github.com/zabbix/zabbix-docker/blob/3.4/web-nginx-mysql/ubuntu/conf/etc/zabbix/nginx.conf>

```shell
server {
    listen          80;
    server_name     zabbix;
    index           index.php;

    access_log      /dev/fd/1 main;
    error_log       /dev/fd/2 notice;

    set $webroot '/usr/share/zabbix';

    root $webroot;

    large_client_header_buffers 8 8k;
    client_max_body_size 10M;


    location = /favicon.ico {
        log_not_found off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    # deny running scripts inside writable directories
    location ~* /(images|cache|media|logs|tmp)/.*\.(php|pl|py|jsp|asp|sh|cgi)$ {
        return 403;
        error_page 403 /403_error.html;
    }

    # Deny all attempts to access hidden files such as .htaccess, .htpasswd, .DS_Store (Mac).
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

    # caching of files
    location ~* \.(ico|pdf|flv)$ {
        expires 1y;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|swf|xml|txt)$ {
        expires 14d;
    }

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ .php$ {
        fastcgi_pass   unix:/var/run/php/php7.2-fpm.sock;
        fastcgi_index  index.php;

        fastcgi_param  SCRIPT_FILENAME  $webroot$fastcgi_script_name;

        include fastcgi_params;
        fastcgi_param  QUERY_STRING     $query_string;
        fastcgi_param  REQUEST_METHOD   $request_method;
        fastcgi_param  CONTENT_TYPE     $content_type;
        fastcgi_param  CONTENT_LENGTH   $content_length;
        fastcgi_intercept_errors        on;
        fastcgi_ignore_client_abort     off;
        fastcgi_connect_timeout 60;
        fastcgi_send_timeout 180;
        fastcgi_read_timeout 180;
        fastcgi_buffer_size 128k;
        fastcgi_buffers 4 256k;
        fastcgi_busy_buffers_size 256k;
        fastcgi_temp_file_write_size 256k;
    }
}
```

zabbix_ssl.conf <https://github.com/zabbix/zabbix-docker/blob/3.4/web-nginx-mysql/ubuntu/conf/etc/zabbix/nginx_ssl.conf>

```shell
server {
    listen          443 ssl http2;
    server_name     zabbix;
    server_name_in_redirect off;

    index  index.php;
    access_log      /dev/fd/1 main;
    error_log       /dev/fd/2 error;

    set $webroot '/usr/share/zabbix';

    root $webroot;

    large_client_header_buffers 8 8k;

    client_max_body_size 10M;


    ssl on;
#    ssl_stapling on;
    ssl_certificate     /etc/ssl/nginx/ssl.crt;
    ssl_certificate_key /etc/ssl/nginx/ssl.key;
    ssl_dhparam /etc/ssl/nginx/dhparam.pem;

    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_verify_depth 3;
    ssl_session_cache    shared:SSL:10m;
    ssl_session_timeout  10m;
    ssl_prefer_server_ciphers on;

    add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";
    add_header Content-Security-Policy-Report-Only "default-src https:; script-src https: 'unsafe-eval' 'unsafe-inline'; style-src https: 'unsafe-inline'; img-src https: data:; font-src https: data:; report-uri /csp-report";

    location =/nginx_status {
        stub_status on;
        access_log   off;
        allow 127.0.0.1;
        deny all;
    }

    location = /favicon.ico {
        log_not_found off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    # deny running scripts inside writable directories
    location ~* /(images|cache|media|logs|tmp)/.*\.(php|pl|py|jsp|asp|sh|cgi)$ {
        return 403;
        error_page 403 /403_error.html;
    }

    # Deny all attempts to access hidden files such as .htaccess, .htpasswd, .DS_Store (Mac).
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

    # caching of files
    location ~* \.(ico|pdf|flv)$ {
        expires 1y;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|swf|xml|txt)$ {
        expires 14d;
    }

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ .php$ {
        fastcgi_pass   unix:/var/run/php/php7.2-fpm.sock;
        fastcgi_index  index.php;

        fastcgi_param  SCRIPT_FILENAME  $webroot$fastcgi_script_name;

        include fastcgi_params;
        fastcgi_param  QUERY_STRING     $query_string;
        fastcgi_param  REQUEST_METHOD   $request_method;
        fastcgi_param  CONTENT_TYPE     $content_type;
        fastcgi_param  CONTENT_LENGTH   $content_length;
        fastcgi_intercept_errors        on;
        fastcgi_ignore_client_abort     off;
        fastcgi_connect_timeout 60;
        fastcgi_send_timeout 180;
        fastcgi_read_timeout 180;
        fastcgi_buffer_size 128k;
        fastcgi_buffers 4 256k;
        fastcgi_busy_buffers_size 256k;
        fastcgi_temp_file_write_size 256k;
    }
}
```

## Start Zabbix server and agent processes

Start Zabbix server and agent processes and make it start at system boot:

```shell
systemctl restart zabbix-server zabbix-agent
systemctl enable zabbix-server zabbix-agent

systemctl restart zabbix-proxy zabbix-agent
systemctl enable zabbix-proxy zabbix-agent
```

## Reference

* [Download and install Zabbix - Official Site](https://www.zabbix.com/download?zabbix=3.4&os_distribution=debian&os_version=stretch&db=MySQL)
* [Installation from sources](https://www.zabbix.com/documentation/3.4/manual/installation/install)
* [Installation from packages - Debian/Ubuntu](https://www.zabbix.com/documentation/3.4/manual/installation/install_from_packages/debian_ubuntu)
* [Database creation](https://www.zabbix.com/documentation/3.4/manual/appendix/install/db_scripts)
* [Installation from containers](https://www.zabbix.com/documentation/3.4/manual/installation/containers)
* [Official Zabbix Dockerfiles - GitHub](https://github.com/zabbix/zabbix-docker)
