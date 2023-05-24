# Telegraf, InfluxDB and Grafana with Docker

This document explains how to setup Grafana with Influx DB using docker containers

## How to step-by-step guide

### Preparation

1. Update and upgrade ubuntu -> `sudo apt update && sudo apt upgrade`
1. If ubuntu desktop install ssh `sudo apt install ssh`
1. Set a static ip address
2. Clone this repository `git clone https://github.com/Ensign-College/tig-docker.git`

### Telegraf, InfluxDB and Grafana with Docker

1. Install docker on a Linux Machine: https://docs.docker.com/engine/install/ubuntu/
3. Copy `.env.example` file to your local enviroment
4. Rename `.env.example` file to `.env`
5. Replace all necessary enviromental variables with the proper configuration values (credentials, bucket names, etc)
6. Run `docker compose up -d` or `docker compose up` to build and run all services together
7. Check that all services are running `docker ps`

### Nginx

2. Install `nginx` -> `sudo apt install nginx`
3. Create the directory `sudo mkdir /etc/nginx/ssl`
3. remove the default nginx config file `sudo rm /etc/nginx/sites-enabled/default`
3. Copy `default.conf.example` file to your local enviroment `cp default.conf.example /etc/nginx/sites-enabled/default.conf`
4. Edit the new nginx `default.conf` file -> `sudo nano /etc/nginx/sites-enabled/default.conf` and replace the IP Address with the right one
5. Generate ssl certificates running `sudo openssl req -x509 -nodes -days 3652 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx-selfsigned.key -out /etc/nginx/ssl/nginx-selfsigned.crt -config ssl-info.txt`
7. Enable `nginx` -> `sudo service nginx enable`
7. Enable `nginx` -> `sudo service nginx restart`
7. Enable `nginx` -> `sudo service nginx status`

### Testing
- Test the grafana accessng through `https://[IPADDRESS]`
- Test the influxdb accessng through `http://[IPADDRESS]:8086`
