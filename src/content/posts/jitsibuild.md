---
title: 开源视频会议项目Jitsi Meet搭建记录
published: 2026-08-24
description: docker部署Jitsi Meet记录
tags: [docker, vps]
category: VPS
slug: jitsi-meet-build
draft: false
---
今日份的业务需求是《给系统集成一个视频会议功能》。  
[Self-Hosting Guide - Docker](https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-docker)这个官方文档写的还是不错的，可以直接照着来。  
配置参考下图，正常的4核8g肯定够了。测试了小型4人会议开启屏享的情况下，2g应该就够了。
![](https://img.koisukui.top/file/jitsimeetbuild/1787648982754_image.png)

# 下载与配置
可以直接根据官网的操作来，最新的国内没找到镜像，就不拉最新的了
```bash
wget $(wget -q -O - https://api.github.com/repos/jitsi/docker-jitsi-meet/releases/tags/stable-11031 | grep zip | cut -d\" -f4)
```
然后解压
```bash
unzip <filename>
cd <foldername>
```
之后照着下面修改一下env文件
```bash
cp env.example .env
vim .env
```
```bash
# shellcheck disable=SC2034

################################################################################
################################################################################
# Welcome to the Jitsi Meet Docker setup!
#
# This sample .env file contains some basic options to get you started.
# The full options reference can be found here:
# https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-docker
################################################################################
################################################################################


#
# Basic configuration options
#

# Directory where all configuration will be stored
CONFIG=~/.jitsi-meet-cfg

# Exposed HTTP port (will redirect to HTTPS port)
HTTP_PORT=8011

# Exposed HTTPS port
HTTPS_PORT=8443

# System time zone
TZ=Asia/Shanghai

# Public URL for the web service (required)
# Keep in mind that if you use a non-standard HTTPS port, it has to appear in the public URL
PUBLIC_URL=https://yourdomain.example

# Media IP addresses and ports to advertise by the JVB
# This setting deprecates DOCKER_HOST_ADDRESS, and supports a comma separated list of IPs
# See the "Running behind NAT or on a LAN environment" section in the Handbook:
# https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-docker#running-behind-nat-or-on-a-lan-environment
#JVB_ADVERTISE_IPS=192.168.1.1,1.2.3.4,192.168.178.1#12000,fe80::1#12000

#
# Memory limits for Java components
#

#JICOFO_MAX_MEMORY=3072m
#VIDEOBRIDGE_MAX_MEMORY=3072m

#
# JaaS Components (beta)
# https://jaas.8x8.vc
#

# Enable JaaS Components (hosted Jigasi)
# NOTE: if Let's Encrypt is enabled a JaaS account will be automatically created, using the provided email in LETSENCRYPT_EMAIL
#ENABLE_JAAS_COMPONENTS=0

#
# Let's Encrypt configuration
#

# Enable Let's Encrypt certificate generation
#ENABLE_LETSENCRYPT=1

# Domain for which to generate the certificate
#LETSENCRYPT_DOMAIN=meet.example.com

# E-Mail for receiving important account notifications (mandatory)
#LETSENCRYPT_EMAIL=alice@atlanta.net

# Use the staging server (for avoiding rate limits while testing)
#LETSENCRYPT_USE_STAGING=1

# Set ACME server. Default is zerossl, you can peek one at https://github.com/acmesh-official/acme.sh/wiki/Server
#LETSENCRYPT_ACME_SERVER="letsencrypt"

#
# Etherpad integration (for document sharing)
#

# Set the etherpad-lite URL in the docker local network (uncomment to enable)
#ETHERPAD_URL_BASE=http://etherpad.meet.jitsi:9001

# Set etherpad-lite public URL, including /p/ pad path fragment (uncomment to enable)
#ETHERPAD_PUBLIC_URL=https://etherpad.my.domain/p/

#
# rtcstats integration
#

# Enable rtcstats analytics (uncomment to enable)
#RTCSTATS_ENABLED=true

# Send the console logs to the rtcstats server
#RTCSTATS_STORE_LOGS=true

# The interval at which rtcstats will poll getStats, defaults to 10000ms.
#RTCSTATS_POLL_INTERVAL=10000

# Send the SDP to the rtcstats server
#RTCSTATS_SEND_SDP=true

#
# Whiteboard integration
#

# Set the excalidraw-backend URL in the docker local network (uncomment to enable)
#WHITEBOARD_COLLAB_SERVER_URL_BASE=http://whiteboard.meet.jitsi

# Set the excalidraw-backend public URL (uncomment to enable)
#WHITEBOARD_COLLAB_SERVER_PUBLIC_URL=https://whiteboard.meet.my.domain


#
# Basic Jigasi configuration options (needed for SIP gateway support)
#

# SIP URI for incoming / outgoing calls
#JIGASI_SIP_URI=test@sip2sip.info

# Password for the specified SIP account as a clear text
#JIGASI_SIP_PASSWORD=

# SIP server (use the SIP account domain if in doubt)
#JIGASI_SIP_SERVER=sip2sip.info

# SIP server port
#JIGASI_SIP_PORT=5060

# SIP server transport
#JIGASI_SIP_TRANSPORT=UDP


#
# Authentication configuration (see handbook for details)
#

# Enable authentication (will ask for login and password to join the meeting)
#ENABLE_AUTH=1

# Enable guest access (if authentication is enabled, this allows for users to be held in lobby until registered user lets them in)
ENABLE_GUESTS=1

# Select authentication type: internal, jwt, ldap or matrix
#AUTH_TYPE=internal

# JWT authentication
#

# Application identifier
#JWT_APP_ID=my_jitsi_app_id

# Application secret known only to your token generator
#JWT_APP_SECRET=my_jitsi_app_secret

# (Optional) Set asap_accepted_issuers as a comma separated list
#JWT_ACCEPTED_ISSUERS=my_web_client,my_app_client

# (Optional) Set asap_accepted_audiences as a comma separated list
#JWT_ACCEPTED_AUDIENCES=my_server1,my_server2

# LDAP authentication (for more information see the Cyrus SASL saslauthd.conf man page)
#

# LDAP url for connection
#LDAP_URL=ldaps://ldap.domain.com/

# LDAP base DN. Can be empty
#LDAP_BASE=DC=example,DC=domain,DC=com

# LDAP user DN. Do not specify this parameter for the anonymous bind
#LDAP_BINDDN=CN=binduser,OU=users,DC=example,DC=domain,DC=com

# LDAP user password. Do not specify this parameter for the anonymous bind
#LDAP_BINDPW=LdapUserPassw0rd

# LDAP filter. Tokens example:
# %1-9 - if the input key is user@mail.domain.com, then %1 is com, %2 is domain and %3 is mail
# %s - %s is replaced by the complete service string
# %r - %r is replaced by the complete realm string
#LDAP_FILTER=(sAMAccountName=%u)

# LDAP authentication method
#LDAP_AUTH_METHOD=bind

# LDAP version
#LDAP_VERSION=3

# LDAP TLS using
#LDAP_USE_TLS=1

# List of SSL/TLS ciphers to allow
#LDAP_TLS_CIPHERS=SECURE256:SECURE128:!AES-128-CBC:!ARCFOUR-128:!CAMELLIA-128-CBC:!3DES-CBC:!CAMELLIA-128-CBC

# Require and verify server certificate
#LDAP_TLS_CHECK_PEER=1

# Path to CA cert file. Used when server certificate verify is enabled
#LDAP_TLS_CACERT_FILE=/etc/ssl/certs/ca-certificates.crt

# Path to CA certs directory. Used when server certificate verify is enabled
#LDAP_TLS_CACERT_DIR=/etc/ssl/certs

# Wether to use starttls, implies LDAPv3 and requires ldap:// instead of ldaps://
# LDAP_START_TLS=1


#
# Security
#
# Set these to strong passwords to avoid intruders from impersonating a service account
# The service(s) won't start unless these are specified
# Running ./gen-passwords.sh will update .env with strong passwords
# You may skip the Jigasi and Jibri passwords if you are not using those
# DO NOT reuse passwords
#

# XMPP password for Jicofo client connections
JICOFO_AUTH_PASSWORD=

# XMPP password for JVB client connections
JVB_AUTH_PASSWORD=

# XMPP password for Jigasi MUC client connections
JIGASI_XMPP_PASSWORD=

# XMPP password for Jigasi transcriber client connections
JIGASI_TRANSCRIBER_PASSWORD=

# XMPP recorder password for Jibri client connections
JIBRI_RECORDER_PASSWORD=

# XMPP password for Jibri client connections
JIBRI_XMPP_PASSWORD=

#
# Docker Compose options
#

# Container restart policy
#RESTART_POLICY=unless-stopped

# Jitsi image registry/namespace (defaults to ghcr.io/jitsi)
#JITSI_IMAGE_REPO=ghcr.io/jitsi

# Local Jitsi image version (useful for local development)
#JITSI_IMAGE_REPO=jitsi
#JITSI_IMAGE_VERSION=latest
```
其中要修改的是`PUBLIC_URL`，这个改成自己的域名，`ENABLE_GUESTS`允许访客访问，方便一会测试。`HTTP_PORT`和`HTTPS_PORT`只需要确保端口没被占用就行了。  
保存退出后，生成一下强密钥。检查一下env里那几个带`_PASSWORD`的字段是否填上了。
```bash
./gen-passwords.sh
```
最后在宿主机上创建一下目录挂载容器的数据。
```bash
mkdir -p ~/.jitsi-meet-cfg/{web,prosody/config,prosody/prosody-plugins-custom,jicofo,jvb,jigasi,jibri,transcriber}
mkdir -p ~/.jitsi-meet-cfg/storage/{jibri,prosody,transcripts,web}
mkdir -p ~/.jitsi-meet-cfg/tmp/{web-crontabs,web-load-test}
chmod 777 ~/.jitsi-meet-cfg/storage/{jibri,prosody,transcripts,web}
chmod 777 ~/.jitsi-meet-cfg/tmp/{web-crontabs,web-load-test}
```

# 反代
先放行一下如下的端口
![](https://img.koisukui.top/file/jitsimeetbuild/1787295084124_image.png)
然后去配置一下域名的解析，类型为A，cf的话记得关闭小黄云。  
由于必须要是https请求，所以还得申请一下ssl证书。`Let’s Encrypt`的教程一大堆，这里就简单带过一下了。
先安装一下certbot，这台是腾讯云的OpenCloudOS。
```bash
sudo dnf install certbot -y
sudo dnf install python3-certbot-nginx -y
```
然后申领一下
```bash
sudo certbot --nginx -d yourdomain.example
```
当然如果不是通过系统包管理器安装的Nginx，比如一些面板安装的，那默认路径就会报错，可以用`standalone`，一会手动配置Nginx。运行前需要先停止正在运行的Nginx。
```bash
sudo certbot certonly --standalone -d yourdomain.example
```
如果是系统安装的，只需要在server块中添加下面配置即可。
```nginx
    location /xmpp-websocket {
    proxy_pass http://127.0.0.1:8011;  #env中的HTTP_PORT地址
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    }

    location /colibri-ws {
        proxy_pass http://127.0.0.1:8011;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
    
    location / {
        proxy_pass http://127.0.0.1:8011;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
```
面板的话，先去添加一下站点，然后配置一下ssl，以宝塔为例。
```bash
cat /etc/letsencrypt/live/yourdomain.example/fullchain.pem
cat /etc/letsencrypt/live/yourdomain.example/privkey.pem
```
![](https://img.koisukui.top/file/jitsimeetbuild/1787301552637_image.png)
fullchain填到证书中，privkey填到密钥中。配置文件中注释掉下面的这些代码，然后把前面的配置加上即可。
```nginx
    # location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    # {
    #     expires      30d;
    #     error_log /dev/null;
    #     access_log /dev/null;
    # }

    # location ~ .*\.(js|css)?$
    # {
    #     expires      12h;
    #     error_log /dev/null;
    #     access_log /dev/null;
    # }
```
同时，由于面板本身会占用一个端口号，就可能会冲突，例如宝塔面板的8888端口就会和jicofo冲突。
这时可以在env文件中新增一行`JICOFO_REST_PORT=8889`，换一个端口，或者直接在compose里面改。  
到这里还有一个问题，当我们发起http请求时，jitsi容器内的Nginx会返回`301 https://yourdomain.example`。接收到请求后又会通过宿主机的Nginx代理发起http请求，那就循环了。所以需要在env文件中再加上一行`ENABLE_HTTP_REDIRECT=0`  
最后启动docker就好了。
```bash
docker compose up -d
```