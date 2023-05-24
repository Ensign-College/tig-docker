# Telegraf, InfluxDB and Grafana with Docker

This document explains how to setup Grafana with Influx DB using docker containers and configuring reverse proxy to access the grafana dashboard over HTTPS.
This guide was tested using the followins OSs/Images:
- Ubuntu Desktop 22.04.2 LTS or Ubuntu Server 22.04.2 LTS
- Docker images:
    - influxdb 2.7.1
    - telegraf 1.26.2
    - grafana/grafana-oss 8.4.3

## How to step-by-step guide

### Preparation

1. Update and upgrade ubuntu -> `sudo apt update && sudo apt upgrade`
2. If ubuntu desktop install ssh `sudo apt install ssh`
3. If ubuntu desktop install git `sudo apt install git`
4. Set a static ip address
5. Clone this repository `git clone https://github.com/Ensign-College/tig-docker.git`
6. Make sure that you are working in the root directory of the cloned repository, this will most commonly be on `/home/[USER]/tig-docker`. To change to that directory run `cd /home/[USER]/tig-docker`

### Telegraf, InfluxDB and Grafana with Docker

1. Install docker on a Linux Machine: https://docs.docker.com/engine/install/ubuntu/
2. After the installation is complete make docker to run with a non-root user. Use the following guide https://docs.docker.com/engine/install/linux-postinstall/
3. Copy `.env.example` file to your local enviroment
4. Rename `.env.example` file to `.env`
5. Replace all necessary enviromental variables with the proper configuration values (credentials, bucket names, etc)
6. Run `docker compose up -d` or `docker compose up` to build and run all services together
7. Check that all services are running `docker ps`

### Nginx

1. Install `nginx` -> `sudo apt install nginx`
2. Create the directory `sudo mkdir /etc/nginx/ssl`
3. remove the default nginx config file `sudo rm /etc/nginx/sites-enabled/default`
4. Copy `default.conf.example` file to your local enviroment `sudo cp default.conf.example /etc/nginx/sites-enabled/default.conf`
5. Edit the new nginx `default.conf` file -> `sudo nano /etc/nginx/sites-enabled/default.conf` and replace the IP Address with the right one
6. Generate ssl certificates running `sudo openssl req -x509 -nodes -days 3652 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx-selfsigned.key -out /etc/nginx/ssl/nginx-selfsigned.crt -config ssl-info.txt`
7. Enable `nginx` -> `sudo systemctl enable nginx`
8. Restart `nginx` -> `sudo service nginx restart`
9. Check if nginx accept changes and if is running correctly `nginx` -> `sudo service nginx status`

### Testing
- Test the grafana accessng through `https://[IPADDRESS]`
- Test the influxdb accessng through `http://[IPADDRESS]:8086`

# Coming soon...
- Grafana dashboard configuration
- Connecting influxdb bucket to grafana dashboard
- Creating, updating or customizing InfluxDB queries
- Set InfluxDB on nginx for HTTPS traffic
