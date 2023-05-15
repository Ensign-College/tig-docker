# Telegraf, InfluxDB and Grafana with Docker

This document explains how to setup Grafana with Influx DB using docker containers

## How to step-by-step guide

1. Install docker on a Linux Machine: https://docs.docker.com/engine/install/ubuntu/
2. Clone this repository
3. Copy `.env.example` file to your local enviroment
4. Rename `.env.example` file to `.env`
5. Replace all necessary enviromental variables with the proper configuration values
6. Run `docker compose up -d` or `docker compose up` to build and run all services together
