---
title: "使用OpenSSL1.1.1编译nginx以支持TLSv1.3"
categories:
  - nginx
tags:
  - nginx
  - TLSv1.3
  - OpenSSL 1.1.1 LTS
---

09月11日OpenSSL 1.1.1 LTS(Long Term Support长期支持版)[正式发布](https://www.openssl.org/blog/blog/2018/09/11/release111/)，~~是时候启用TLSv1.3了~~。刚好Chrome 69正式版和Firefox 62正式版都发布了，但只支持到草案28。各版本浏览器TLSv1.3[支持情况](https://caniuse.com/#feat=tls1-3)。目前Chrome Beta和Firefox Nightly可以手动启用正式版的TLSv1.3的支持。

## 安装编译需要的工具

```shell
apt-get install build-essential libpcre3-dev zlib1g-dev checkinstall
```

## 下载支持TLSv1.3 final RFC version(RFC 8446)的OpenSSL1.1.1源码

```shell
cd /usr/local/src
wget https://www.openssl.org/source/openssl-1.1.1.tar.gz
tar -zxf openssl-1.1.1.tar.gz
# cd openssl-1.1.1
# ./config --prefix=/usr/local/ssl --openssldir=/usr/local/ssl -Wl,--enable-new-dtags,-rpath,/usr/local/ssl/lib
# make
# make install
```

## 下载nginx1.15.3源码

```shell
wget https://nginx.org/download/nginx-1.15.3.tar.gz
tar zxf nginx-1.15.3.tar.gz
cd nginx-1.15.3
./configure \
--prefix=/etc/nginx \
--sbin-path=/usr/sbin/nginx \
--modules-path=/usr/lib/nginx/modules \
--conf-path=/etc/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx.pid \
--lock-path=/var/run/nginx.lock \
--http-client-body-temp-path=/var/cache/nginx/client_temp \
--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
--http-scgi-temp-path=/var/cache/nginx/scgi_temp \
--user=nginx \
--group=nginx \
--with-compat \
--with-file-aio \
--with-threads \
--with-http_addition_module \
--with-http_auth_request_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_mp4_module \
--with-http_random_index_module \
--with-http_realip_module \
--with-http_secure_link_module \
--with-http_slice_module \
--with-http_ssl_module \
--with-http_stub_status_module \
--with-http_sub_module \
--with-http_v2_module \
--with-mail \
--with-mail_ssl_module \
--with-stream \
--with-stream_realip_module \
--with-stream_ssl_module \
--with-stream_ssl_preread_module \
--with-cc-opt='-g -O2 -fdebug-prefix-map=/data/builder/debuild/nginx-1.15.3/debian/debuild-base/nginx-1.15.3=. -specs=/usr/share/dpkg/no-pie-compile.specs -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -fPIC' \
--with-ld-opt='-specs=/usr/share/dpkg/no-pie-link.specs -Wl,-z,relro -Wl,-z,now -Wl,--as-needed -pie' \
--with-openssl=/usr/local/src/openssl-1.1.1
# 卸载旧版本，保留配置
# apt-get remove nginx
make
checkinstall
# mkdir /var/log/nginx
# mkdir /var/cache/nginx
```

## 配置nginx

添加TLSv1.3和密码套件

* TLS_AES_256_GCM_SHA384
* TLS_CHACHA20_POLY1305_SHA256
* TLS_AES_128_GCM_SHA256
* TLS_AES_128_CCM_8_SHA256
* TLS_AES_128_CCM_SHA256

默认情况下，上述密码套件中的前三个默认启用。

```shell
ssl_protocols TLSv1.3;
ssl_ciphers 'TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256';
```

## 开机启动nginx

在/etc/systemd/system新建nginx.service文件，复制以下内容

```shell
# Stop dance for nginx
# =======================
#
# ExecStop sends SIGSTOP (graceful stop) to the nginx process.
# If, after 5s (--retry QUIT/5) nginx is still running, systemd takes control
# and sends SIGTERM (fast shutdown) to the main process.
# After another 5s (TimeoutStopSec=5), and if nginx is alive, systemd sends
# SIGKILL to all the remaining processes in the process group (KillMode=mixed).
#
# nginx signals reference doc:
# http://nginx.org/en/docs/control.html
#
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

设置开机启动

```shell
systemctl enable nginx
```

## 浏览器开启TLSv1.3支持

* Chrome Canary版本默认开启，其它版本如未开启可在地址栏输入`chrome://flags/#tls13-variant`修改其值为`Enabled (Final)`以支持最新TLSv1.3正式版
* Firefox Nightly版本默认开启，如未开启可到`about:config`将`security.tls.version.max`修改为`4`

## 参考

* [OpenSSL 1.1.1 Is Released - OpenSSL Blog](https://www.openssl.org/blog/blog/2018/09/11/release111/)
* [TLS1.3 - OpenSSLWiki](https://wiki.openssl.org/index.php/TLS1.3)
* [Building nginx from Sources - nginx documentation](https://nginx.org/en/docs/configure.html)
* [CheckInstall - Debian Wiki](https://wiki.debian.org/CheckInstall)
* [CheckInstall - Community Help Wiki](https://help.ubuntu.com/community/CheckInstall)
* [本博客开始支持 TLS 1.3 - JerryQu 的小站](https://imququ.com/post/enable-tls-1-3.html)
