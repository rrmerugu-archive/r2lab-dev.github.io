---
layout: post
title:  My first homelab setup 
date:   2022-12-03
categories: ["How-to", ]
story: true
Keywords: ["homelab", ]
---

## Install docker

```bash
sudo apt-get update && sudo apt-get upgrade -y
 
curl -fsSL test.docker.com -o get-docker.sh && sh get-docker.sh
sudo usermod -aG docker ${USER}
docker run hello-world
logout
```
Now logout and login again.

### Install docker-compose
```shell
sudo apt-get install libffi-dev libssl-dev
sudo apt install python3-dev
sudo apt-get install -y python3 python3-pip
sudo pip3 install docker-compose
```

### start docker on start-up
```shell
sudo systemctl enable docker
```

## Setting up SSH 

First off, change the password using the command `passwd`. 
```bash
pi@box:~ $ passwd
Changing password for pi.
Current password: 
New password: 
Retype new password: 
passwd: password updated successfully
```

To enable Public Key Authentication, uncomment `PubkeyAuthentication` and update following in `/etc/ssh/sshd_config`  

```shell
PubkeyAuthentication yes
PasswordAuthentication no
```

Restart the ssh service for the changes to take effect.
```shell
sudo service ssh restart
```
 


## Setup Reverse Proxy

```
#docker-compose.yml
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ~/storage/npm/data:/data
      - ~/storage/npm/letsencrypt:/etc/letsencrypt
```

## Dashboard


```bash

docker run -d \
  --name=heimdall \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Asia/Kolkata \
  -p 12080:80 \
  -p 12443:443 \
  -v ~/storage/heimdall/config:/config \
  --restart unless-stopped \
  lscr.io/linuxserver/heimdall:latest

```
## Password Manager

Setup vaultwarden


For gmail as SMLTP https://github.com/dani-garcia/vaultwarden/issues/1131#issuecomment-691102235
https://github.com/dani-garcia/vaultwarden/wiki/Enabling-WebSocket-notifications
```shell
docker run -d --name vaultwarden \
  -v ~/storage/vaultwarden/data:/data/ \
  --restart unless-stopped \
  -e WEBSOCKET_ENABLED=true \
  -e SIGNUPS_ALLOWED=false \
  -e DOMAIN=https://vault.r2lab.dev \
  -e ADMIN_TOKEN=admin-token-here \
  -e EMERGENCY_ACCESS_ALLOWED=true \
  -e SEND_ALLOWED=true \
  -e WEB_VAULT_ENABLED=true \
  -p 12180:80 -p 3012:3012 \
  vaultwarden/server:latest
```

```
-e SIGNUPS_VERIFY=true \
-e SIGNUPS_VERIFY_RESEND_time=3600 \
-e SIGNUPS_VERIFY_RESEND_LIMIT=6 \

export SMTP_HOST=smtp.gmail.com
export SMTP_FROM=ab@gmail.com
export SMTP_SSL=true
export SMTP_PORT=587
export SMTP_USERNAME=gmailcom
export SMTP_PASSWORD=password
-e SMTP_HOST=${SMTP_HOST} \
-e SMTP_FROM=${SMTP_FROM} \
-e SMTP_PORT=${SMTP_PORT} \
-e SMTP_SSL=${SMTP_SSL} \
-e SMTP_USERNAME=${SMTP_USERNAME} \
-e SMTP_PASSWORD=${SMTP_PASSWORD} \


// npm
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
`
  ```

## References

1. https://www.geekyhacker.com/2021/02/15/configure-ssh-key-based-authentication-on-raspberry-pi/
2. https://dev.to/elalemanyo/how-to-install-docker-and-docker-compose-on-raspberry-pi-1mo