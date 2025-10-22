#!/bin/bash
############################
# Commands to install vps-1af3a49e
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

#use the file included in this git repo
sudo nano /etc/nginx/sites-available/default

sudo nginx -t
sudo systemctl reload nginx
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'
sudo certbot --nginx -d oicrt.com
sudo certbot --nginx -d fundamentalhistory.com

# Verify your certbot auto renewal
sudo systemctl status certbot.timer
sudo certbot renew --dry-run

