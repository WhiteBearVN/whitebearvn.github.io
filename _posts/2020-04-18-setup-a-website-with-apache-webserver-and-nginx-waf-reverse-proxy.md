---
title: Set up website with apache webserver and nginx waf reverse proxy
author: F4c3l3ss_
date: 2020-04-03
categories: [Research]
tags: [nginx,forensics,apache,webserver,system,waf]
math: true
mermaid: true
---

When I started thinking about running my site on my own server instead of using Blogger or Medium, etc… And after tried to use Apache, Nginx,… I realized that combining Apache and Nginx could be the best choice for me, because I want to use ```.htaccess``` of Apache and speed of Nginx. There come my architecture:

![IMG](/assets/img/blog/Diagram.jpg)

In this post, I will use private IP range for Nginx reverse proxy server and Apache web server. Both of them are running Ubuntu 18.04 LTS.

* Apache webserver: 	10.128.0.2

* Nginx reverse proxy: 	10.128.0.3

# 1. Install and configure Apache web server
 
Install apache webserver:

```bash
$ sudo apt install apache2
```

Next we create a place where to storage source code:

```bash
$ sudo mkdir -p /var/www/html/example.com/public_html
```

Change permission:

```bash
$ sudo chown -R <user>:<group> /var/www/html/example.com/public_html
$ sudo chmod -R 744 /var/www/html/example.com/public_html
```

With **user** and **group* you can use command 

```bash
$ apachectl -S
```

Usually, it's www-data

Now let's config virutal host and port:

```bash
$ sudo vim /etc/apache2/ports.conf

Listen 8080

<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>

$ sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/example.com.conf
$ sudo vim /etc/apache2/sites-available/example.com.conf

<VirtualHost 10.128.0.2:8080>
        ServerAdmin webmaster@example.com
        DocumentRoot /var/www/html/example.com/public_html/

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        ErrorDocument 404 /404.html
</VirtualHost>
```

Active your website:

```bash
$ sudo a2dissite 000-default.conf
$ sudo a2ensite example.com.conf
```

# 2. Install and configure Nginx
## 2.1 Install Nginx package

Install the prerequisites:

```bash
$ sudo apt install curl gnupg2 ca-certificates lsb-release
```

Add repository for stable nginx packages:

```bash
$ echo "deb http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

Import an official nginx signing key so apt could verify the packages authenticity:

```bash
$ curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
```

Verify that you now have the proper key:

```bash
$ sudo apt-key fingerprint ABF5BD827BD9BF62
```
The output should contain the full fingerprint 573B FD6B 3D8F BC64 1079 A6AB ABF5 BD82 7BD9 BF62 as follows:

```bash
pub   rsa2048 2011-08-19 [SC] [expires: 2024-06-14]
      573B FD6B 3D8F BC64 1079  A6AB ABF5 BD82 7BD9 BF62
uid   [ unknown] nginx signing key <signing-key@nginx.com>
```

Finally, install Nginx:

```bash
$ sudo apt update
$ sudo apt install nginx
```
## 2.2 Configure Nginx reverse proxy

```bash
$ sudo cp /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/example.com.conf
$ sudo vim /etc/nginx/conf.d/example.com.conf
server {

    listen       80;
    server_name example.com;
	port_in_redirect off;

    location / {
        proxy_redirect off;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://10.128.0.2:8080;
    }

    location ~ /\.ht {
        deny  all;
    }
}
```

## 2.3 Install and configure Modsecurity

Install the prerequisites:

```bash
$ sudo apt-get install -y apt-utils autoconf automake build-essential git libcurl4-openssl-dev libgeoip-dev liblmdb-dev libpcre++-dev libtool libxml2-dev libyajl-dev pkgconf wget zlib1g-dev
```

Download Modsecurity source:

```bash
$ git clone --depth 1 -b v3/master --single-branch https://github.com/SpiderLabs/ModSecurity
```

Compile Modsecurity:

```bash
$ cd ModSecurity
$ git submodule init
$ git submodule update
$ ./build.sh
$ ./configure
$ make
$ sudo make install
```

If you see this notification on your terminal, just ignore it and take a coffee because the process might take about 15 minutes to complete.

> fatal: No names found, cannot describe anything.

## 2.4 Download the NGINX Connector for ModSecurity and Compile It as a Dynamic Module

```bash
$ git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git
```

Check your nginx version:

```bash
$ nginx -v

nginx version: nginx/1.17.9
```

Download source code:

```bash
$ cd nginx-1.17.9
$ ./configure --with-compat --add-dynamic-module=../ModSecurity-nginx
$ make modules
$ sudo cp objs/ngx_http_modsecurity_module.so /etc/nginx/modules/
```

Load Nginx Modsecurity Connector by add the load_module following to the top of ```/etc/nginx/nginx.conf```

> load_module modules/ngx_http_modsecurity_module.so;

Configure and Enable Modsecurity:

```bash
$ sudo mkdir /etc/nginx/modsec
$ sudo wget -P /etc/nginx/modsec/ https://raw.githubusercontent.com/SpiderLabs/ModSecurity/v3/master/modsecurity.conf-recommended
$ sudo mv /etc/nginx/modsec/modsecurity.conf-recommended /etc/nginx/modsec/modsecurity.conf
```

Active dropping malicious mode:

```bash
$ sudo sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /etc/nginx/modsec/modsecurity.conf
```

Add modsecurity rule and config your test rule:

```bash
$ sudo vim /etc/nginx/modsec/main.conf

Include "/etc/nginx/modsec/modsecurity.conf"

# Basic test rule

SecRule ARGS:testparam "@contains test" "id:1234,deny,status:403"
```

Then turn on the modsecurity in your server config:

```bash
$ sudo vim /etc/nginx/conf.d/example.com.conf

server {
    # ...
    modsecurity on;
    modsecurity_rules_file /etc/nginx/modsec/main.conf;
}
```

Check everything ok yet by command:

```bash
$ sudo nginx -t
```

If the output are:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

We completed 99%, if there is an error about unicode.mapping, you can fix it by the following commands:

```bash
$ cd /path/to/Modsecurity
$ sudo cp unicode.mapping /etc/nginx/modsec/
```

Check if the waf is active or not by curl. The ```403``` status confirmed that the rule worked:

```bash
$ curl example.com/?testparam=test
<html>
<head><title>403 Forbidden</title></head>
<body bgcolor="white">
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.13.1</center>
</body>
</html>
```
This is the basic configuration, if you have any problem, just let me know and I will help you!

Cheers!

P/S: Thank you GinaTU aka Tứ Diệp Thảo to help me edit this post

