# Telegraf, InfluxDB and Grafana with Docker

![TIG Architecture](/img/tig_architecture.jpg "TIG Architecture")

This document explains how to set up Grafana with Influx DB using docker containers and configuring the reverse proxy to access the Grafana dashboard over HTTPS.
This guide was tested using the following OSs/Images:
- Ubuntu Desktop 22.04.2 LTS or Ubuntu Server 22.04.2 LTS
- Docker images:
    - influxdb 2.7.1
    - telegraf 1.26.2
    - grafana/grafana-oss 8.4.3

## How to step-by-step guide

### Server Preparation

1. Update and upgrade ubuntu -> `sudo apt update && sudo apt upgrade`
2. If ubuntu desktop install ssh `sudo apt install ssh`
3. If ubuntu desktop install git `sudo apt install git`
4. Set a static ip address https://www.freecodecamp.org/news/setting-a-static-ip-in-ubuntu-linux-ip-address-tutorial/, force change `sudo systemctl restart systemd-networkd`. NOTE: if you are using a terminal, follow the next sub-steps
    1. Edit or create the `yaml` network configuration file on `/etc/netplan/` -> `sudo nano /etc/netplan/00-installer-config.yaml`
    2. Example:
        ```yaml
        network:
        ethernets:
          eth0:
            dhcp4: no
            addresses: [192.168.168.1/24]
            gateway4: 192.168.168.1
            nameservers:
              addresses: [8.8.8.8, 8.8.4.4]
        version: 2
        ```
    3. After saving the file, run `sudo netplan apply`
    4. Check IP address: `ip add show`
    5. If changes haven't applied yet, run `sudo systemctl restart systemd-networkd`
5. Clone this repository `git clone https://github.com/Ensign-College/tig-docker.git`
6. Make sure you are working in the root directory of the cloned repository. This will most commonly be on `/home/[USER]/tig-docker`. To change to that directory, run `cd /home/[USER]/tig-docker`

### ProxMox nodes preparation

ProxMox node data should be collected through the official proxmox
telegraf input plugin. We need to create a user with permissions and an api token for each ProxMox cluster to be able to retrieve VMs and Containers data from each ProxMox node.

On the ProxMox cluster:
1. Connect through ssh to one of the ProxMox nodes (`ssh root@PROXMOX_NODE_IP_ADDRESS`) and enter the corresponding password
2. Run the following command to create api user with permissions, save the generated token
    ```bash
    ## Create a influx user with PVEAuditor role
    pveum user add influx@pve
    pveum acl modify / -role PVEAuditor -user influx@pve
    ## Create a token with the PVEAuditor role
    pveum user token add influx@pve monitoring -privsep 1
    pveum acl modify / -role PVEAuditor -token 'influx@pve!monitoring'
    ```
**NOTE:** This process must be repeated in each of the desired clusters to be monitored


### Telegraf configuration

![Telegraf Main Diagram](/img/telegraf_main_diagram.png "Telegraf Main Diagram")

The preconfiguration is `/telegraf/telegraf.conf`. When the docker container is built, it copies this file from the host machine inside the Telegraf docker container setting up our custom configuration for ProxMox servers. If you want to customize these configurations, you need to edit this file and build the container again.

It needs to be set up at least 1 input in the telegraf config file so that telegraf works appropriately, the same input plugin type can be added multiple times (this is how all ProxMox nodes can be monitored in the same Grafana dashboard).

```toml
# Provides metrics from Proxmox nodes (Proxmox Virtual Environment > 6.2).
[[inputs.proxmox]]
  ## API connection configuration. The API token was introduced in Proxmox v6.2. Required permissions for user and token: PVEAuditor role on /.
  base_url = "https://{PROXMOX_NODE_IP_ADDRESS}/api2/json" #replace the {PROXMOX_NODE_IP_ADDRESS} with the real one
  api_token = "USER@REALM!TOKENID=UUID" #replace the full api token with the one created in the previous step

  ## Node name, defaults to OS hostname
  ## Unless Telegraf is on the same host as Proxmox, setting this is required
  ## for Telegraf to successfully connect to Proxmox. If not on the same host,
  ## leaving this empty will often lead to a "search domain is not set" error.
  node_name = "PROXMOX_NODE_NAME"

  ## Optional TLS Config
  # tls_ca = "/etc/telegraf/ca.pem"
  # tls_cert = "/etc/telegraf/cert.pem"
  # tls_key = "/etc/telegraf/key.pem"
  ## Use TLS but skip chain & host verification
  insecure_skip_verify = true

  # HTTP response timeout (default: 5s)
  response_timeout = "5s"
```

**NOTE:** You have to add as many sections of `[[inputs.proxmox]]` as proxmox nodes you want or need to configure

### Telegraf, InfluxDB and Grafana with Docker

1. Install docker on a Linux Machine: https://docs.docker.com/engine/install/ubuntu/
2. After the installation is completed, make docker run with a non-root user. Use the following guide https://docs.docker.com/engine/install/linux-postinstall/
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

### Testing and Visualizing Data on InfluxDB

As a good practice, we should test if the Telegraf agent is gathering data into InfluxDB. The following is also the process of creating customized queries.
1. Go to `http://[IPADDRESS]:8086` and log in with the corresponding credentials
2. Go to Buckets; we should see at least 3 buckets. `_monitoring` and `_tasks` are InfluxDB system buckets. And the other bucket should be the bucket where we are putting and retrieving data. NOTE: the probable name for this bucket should be `proxmox`
3. The first column (FROM) is the bucket name. All the next following columns are filters. In the first filter, we could select our input (from the `telegraf.conf`), which is called `proxmox`. This input plugin collects data from each ProxMox VM or container. After selecting filters, click `SUBMIT` to see if everything works in real time. To see what the collected metrics or tags go to https://github.com/influxdata/telegraf/blob/master/plugins/inputs/proxmox/README.md
![InlfuxDB Sample Data](/img/influxdb_sample_data.png "InlfuxDB Sample Data")
4. To see the raw flux query click on `SCRIPT EDITOR`. This will show the query as flux query language. This text could be copied to be used on any Grafana dashboard panel.
![InlfuxDB Raw Query](/img/influxdb_raw_query.png "InlfuxDB Raw Query")

### Connect InfluxDB to Grafana

To visualize InfluxDB data on Grafana. You need to add a data source in Grafana.

In the Grafana Dashboard
1. Go to `Configurations / Data sources` and click on `Add data source`
2. Select `InfluxDB`
3. Enter connection information
    1. Fill in a data source name
    2. In `Query Language` select `Flux`
    3. In `url` enter the URL from the telegraf docker container -> `http://influxdb:8086` (make sure to add the corresponding port)
    4. Check that `Basic auth` is selected, and under `Basic Auth Details` enter the InfluxDB `user` and `password`
    5. In `InfluxDB Details` enter the following information:
        1. `Organization`: the organization that was set up in the environmental variables
        2. `Token`: the token that was set up in the environmental variables
        3. `Default Bucket`: the default bucket that was set up in the environmental variables. It is probably `proxmox`
4. Click on `Save & Test`. You should see two success messages: `Datasource updated` and `3 buckets found`

![Datasource updated](/img/data_source_updated.png "Datasource updated") ![3 buckets found](/img/3_buckets_added.png "3 buckets found")

If more details are needed to connect a data source on Grafana, follow one of these tutorials: https://docs.influxdata.com/influxdb/v1.8/tools/grafana/?t=Flux or https://grafana.com/docs/grafana/latest/datasources/influxdb/

### Grafana dashboard

Once a data source (InfluxDB) is added to Grafana. We could import an existing dashboard to display the data, or we could create a dashboard from scratch.

#### Importing an existing dashboard
1. Select a compatible dashboard for `Flux` language. Go to https://grafana.com/grafana/dashboards/. For this example we are using `ID: 15650` (https://grafana.com/grafana/dashboards/15650-telegraf-influxdb-2-0-flux/)
2. Go to the **`+`** sign and click on `import`.
3. Copy or write the dashboard ID, and click on `load`
4. Edit the dashboard name and folder as wanted, then select the `InfluxDB` database.
5. And after importing the dashboard, we should start seeing data.
![Grafana Imported Dashboard](/img/grafana_imported_dashboard.png "Grafana Imported Dashboard")

#### Create a dashboard from scratch
1. Go to the **`+`** sign and click on `Create`.
2. Click on `Add a new panel`
3. Copy the query from the InfluxDB `SCRIPT EDITOR` (See `Testing and Visualizing Data on InfluxDB` in this guide)
4. Customize this panel on panel on the right side. Click on `apply` to finish
![Grafana Creating Panel](/img/grafana_creating_panel.png "Grafana Creating Panel")
5. Then, we can see the created panel on the new dashboard
![Grafana New Dashboard](/img/grafana_new_dashboard.png "Grafana New Dashboard")

### Testing
- Test the grafana accessing through `https://[IPADDRESS]`
- Test the influxdb accessing through `http://[IPADDRESS]:8086`

# Coming soon...
- Creating metrics alerts (maybe with Grafana, Telegraf, or InfluxDB). This needs to be researched.
- Set InfluxDB on nginx for HTTPS traffic.
