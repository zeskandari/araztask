############ installing nginx ################

yum update -y
sudo yum install epel-release -y
sudo yum install nginx -y
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
sudo systemctl start nginx
sudo systemctl enable nginx
nginx -v
systemctl status nginx
curl http://185.213.167.128
or
Open in a web browser:
http://server_domain_name_or_IP/

############# install mysql ###############

yum install epel-release.noarch 
yum -y install mariadb-server mariadb
systemctl enable mariadb.service
systemctl start mariadb.service
mysql_secure_installation
=>Press ENTER
=>Set New Password
=>Repeat Above Password
=>Choose “y” to disable that user
=>Choose “n” for no
=>Choose “y” for yes
=>Choose “y” for yes


########### installing php ###############

yum install yum-utils -y
sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum --disablerepo="*" --enablerepo="remi-safe" list php[7-9][0-9].x86_64
sudo yum-config-manager --enable remi-php74
sudo yum install php  php-mysql  php-fpm  php-pgsql install php-gd -y 
php --version
vim /etc/php-fpm.d/www.conf

…
; RPM: apache user chosen to provide access to the same directories as httpd
user = nginx   
; RPM: Keep a group allowed to write in log dir.
group = nginx  
;listen = 127.0.0.1:9000

listen = /var/run/php-fpm/php-fpm.sock; 
listen.owner = nginx
listen.group = nginx
listen.mode = 0660

:x!
sudo systemctl start php-fpm
systemctl enable php-fpm.service
sudo systemctl restart php-fpm

Configuring Nginx to Process PHP Pages:
vim /etc/nginx/conf.d/default.conf
server {
    listen       80;
    server_name  185.213.167.128;

    root   /usr/share/nginx/html;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
:x!
systemctl restart nginx
Testing PHP Processing on your Web Server:

sudo chown -R nginx.nginx /usr/share/nginx/html/
vim /usr/share/nginx/html/info.php
<?php phpinfo(); ?>
:x!
http://http://185.213.167.128//info.php

############ Install WordPress on Nginx ############
yum install wget -y
sudo wget https://wordpress.org/latest.tar.gz
sudo mv latest.tar.gz /usr/share/nginx/html
cd /usr/share/nginx/html
sudo tar -zxvf latest.tar.gz
sudo chown -R nginx:nginx /usr/share/nginx/html/wordpress
sudo chmod -R 755 /usr/share/nginx/html/wordpress
vim /etc/nginx/conf.d/default.conf
server {
 listen 80;
 server_name 185.213.167.128;
 
 # note that these lines are originally from the "location /" block
 root /usr/share/nginx/html/wordpress;
 index index.php index.html index.htm;
 
 location / {
 try_files $uri $uri/ =404;
 }
 error_page 404 /404.html;
 error_page 500 502 503 504 /50x.html;
 location = /50x.html {
 root /usr/share/nginx/html;
 }
 
 location ~ \.php$ {
 try_files $uri =404;
 fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
 fastcgi_index index.php;
 fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
 include fastcgi_params;
 }
}
:x!
sudo service php-fpm restart
sudo service nginx restart

for check: http://185.213.167.128/wp-admin/setup-config.php

[root@server1 wordpress]# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 17
Server version: 5.5.68-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database wordpress;

MariaDB [(none)]> grant all on wordpress.* to 'z'@'localhost' identified by 'qazwsx';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all on wordpress.* to 'z'@'185.213.167.128' identified by 'qazwsx';

flush privileges;
exit;


#### installing Postgresql ####
i used version 13 postgres for centos 7 for this:

# Install the repository RPM:
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm


sudo yum install -y postgresql13-server

# Optionally initialize the database and enable automatic start:
sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
sudo systemctl enable postgresql-13
sudo systemctl start postgresql-13

#for check
systemctl status postgresql-10
 
###Download and configure the Fork version of WP4PG in your WordPress directory ###
cd /usr/share/nginx/html
ls
cd wordpress/
cd wp-content/
wget https://downloads.wordpress.org/plugin/postgresql-for-wordpress.1.3.1.zip
unzip postgresql-for-wordpress.1.3.1.zip 
 mv postgresql-for-wordpress/pg4wp Pg4wp
 rm -rf postgresql-for-wordpress
cp Pg4wp/db.php db.php

++++++++++++++++++++++++++++++++++++
1- ### Create the SSL Certificate ###  xnotsuccess
sudo mkdir /etc/ssl/private
sudo chmod 700 /etc/ssl/private
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

Step 2 — Configure Nginx to Use SSL
sudo vim /etc/nginx/conf.d/ssl.conf
server {
    listen 443 http2 ssl;
    listen [::]:443 http2 ssl;

    server_name 192.168.1.106;

    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;
}
:x!

Create a Redirect from HTTP to HTTPS

sudo vim /etc/nginx/conf.d/ssl.conf

server {
    listen 80;
    listen [::]:80;
    server_name your_server_ip;
    return 301 https://$host$request_uri;
}
:x!
nginx -t
sudo systemctl restart nginx

============================================
####nginx load balancer -round robin########
on server1,2 install nginx

on server3-loadbalancer->sudo yum install epel-release -y
sudo yum install nginx -y
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
sudo systemctl start nginx
sudo systemctl enable nginx
nginx -v
systemctl status nginx
cd /etc/nginx/conf.d/load-balancer.conf
# Define whichch servers to include in the load balancing scheme.
# It's best to use the servers' private IPs for better performance and security.
# You can find the private IPs at your UpCloud control panel Network section.

upstream backend {
      server 192.168.0.106;
      server 192.168.0.104;

   }

   # This server accepts all traffic to port 80 and passes it to the upstream.
   # Notice that the upstream name and the proxy_pass need to match.

   server {
      listen 80;

      location / {
          proxy_pass http://backend;
      }
   }
systemctl restart nginx
curl localhost
==================================================
##### install prometheus on loadbalancer server #####
yum install epel-release -y
yum install bash-completion-extras -y
yum install chrony -y 
systemctl enable --now chronyd
systemctl start chronyd
systemctl status chronyd
timedatectl
go site https://prometheus.io/download/ 
wget https://github.com/prometheus/prometheus/releases/download/v2.35.0/prometheus-2.35.0.linux-amd64.tar.gz
useradd -M -s /bin/false -r prometheus
ls /home/
mkdir /etc/prometheus
mkdir /{var/lib,etc}/prometheus
tar -xvf prometheus-2.35.0.linux-amd64.tar.gz
cd prometheus-2.35.0.linux-amd64
ls -l
cp prometheus /usr/local/bin/
cp promtool /usr/local/bin
chown prometheus:prometheus /usr/local/bin/prom*
ls -l
cp -a console* /etc/prometheus/
ls /etc/prometheus/
=>console_libraries  consoles
chown -R prometheus:prometheus /etc/prometheus/
chown prometheus:prometheus /var/lib/prometheus
vim /etc/prometheus/prometheus.yml
global:
  scrape_interval: 10s
scrape_configs:
  - job_name: Prometheus Server
    static_configs:
    - targets: ['localhost:9090']
:x!
vim /usr/lib/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
:x!
systemctl daemon-reload
systemctl enable --now prometheus.service
systemctl status prometheus.service
ss -ltun
yum install lsof -y
lsof -Panp 8722 -itcp   =>this num is status main num
lsof -Panp 8722 -itcp -sTCP:LISTEN
firewall-cmd --add-port=9090/tcp --permanent
firewall-cmd --reload
in browser ->192.168.1.105:9090


server1->install haexporter


--server3-loadbalancer-
install docker:
***install docker on centos7:***
yum -y update                                                                     
yum install -y yum-utils
vim /etc/resolv.conf (nameserver 178.22.122.100 فیلترشگن ست کن )
cd /etc/yum.repos.d
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum-config-manager --disable docker-ce-edge
yum makecache fast
yum install -y docker-ce

 systemctl enable docker   
 systemctl start docker
systemctl status docker

install grapphana:
sudo docker run -d -p 3000:3000 grafana/grafana

install ansibe on server2:
yum install epel-release
yum install -y ansible
ansible --version
cd /home/
mkdir -p ansible/provision
cd ansible/provision/
 >myproject.yml
 mkdir inventory
 mkdir roles
 ls
 cd inventory/
 >anisahosts
 ls
 cd ..
 ls
 cd roles/
 mkdir myproject
 cd myproject/
 mkdir tasks vars files defaults templates handlers meta
 cd defaults/
 >main.yml
 cd ..
 ls
 cd handlers/
 >main.yml
 cd ..
 touch meta/main.yml
 touch tasks/main.yml
 touch templates/main.yml
 touch vars/main.yml
 cd ..
 cd ..
 provision# # araztask
