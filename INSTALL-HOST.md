#!/bin/bash
############################
# Commands to install vps-a449c32d.vps.ovh.ca or oicrt.com
############################
#
# Update
sudo apt-get update
sudo apt-get upgrade
#
# Install Syncthing
sudo apt install syncthing
sudo systemctl enable syncthing@debian.service
sudo systemctl start syncthing@debian.service
#
# Install nginx
sudo apt install nginx
sudo apt install ufw
sudo ufw app list
sudo ufw allow 'Nginx HTTP'
systemctl status nginx
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
#
# Nginx with Let's Encrypt
sudo apt install certbot python3-certbot-nginx

sudo nano /etc/nginx/sites-available/default
server {
    # Listen for requests on your domain/IP address.
    server_name oicrt.com;

    root /var/www/html;

    location / {
        # Proxy all requests to Docker App running on port 3080
        proxy_pass http://localhost:3080;

        # Pass on information about the requests to the proxied service using headers
        proxy_set_header HOST $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

sudo nginx -t
sudo systemctl reload nginx
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'
sudo certbot --nginx -d oicrt.com

# Verify your certbot auto renewal
sudo systemctl status certbot.timer
sudo certbot renew --dry-run

# Password Authentication with Nginx
sudo sh -c "echo -n 'debian:' >> /etc/nginx/.htpasswd"
sudo sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd"
cat /etc/nginx/.htpasswd
sudo apt-get install apache2-utils
sudo htpasswd -c /etc/nginx/.htpasswd debian
# Leave out the -c for any additional users you additional
sudo htpasswd /etc/nginx/.htpasswd another_user
cat /etc/nginx/.htpasswd

# Add to the nginx config for .htpasswd
#sudo nano /etc/nginx/sites-enabled/default
#server {
#    listen 80 default_server;
#    listen [::]:80 default_server ipv6only=on;
#
#    root /usr/share/nginx/html;
#    index index.html index.htm;
#
#    server_name localhost;
#
#    location / {
#        try_files $uri $uri/ =404;
#        auth_basic "Restricted Content";
#        auth_basic_user_file /etc/nginx/.htpasswd;
#    }
#}
sudo service nginx restart

################
### Add docker.oicrt.com
#############
sudo nano /etc/nginx/sites-available/docker
server {
    # Listen for requests on your domain/IP address.
    server_name oicrt.com;

    root /var/www/html;

    location / {
        # Proxy all requests to Docker App running on port 3080
        proxy_pass http://localhost:3080;

        # Pass on information about the requests to the proxied service using headers
        proxy_set_header HOST $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
sudo ln -s /etc/nginx/sites-available/docker /etc/nginx/sites-enabled/docker
# Check the config is good
sudo nginx -t
#Output
#nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
#nginx: configuration file /etc/nginx/nginx.conf test is successful
##########
sudo systemctl restart nginx

sudo certbot --nginx -d oicrt.com

sudo systemctl restart nginx

sudo ln -s /home/debian/oicrt.com/app/html /var/www/html
