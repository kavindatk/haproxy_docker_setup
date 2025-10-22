# Part 4: Docker and HAProxy Setup for 3-NameNode Hadoop-Hive Cluster 

<br/><br/>
<p align="center">
<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_hue_fast_access/blob/main/images/docker.jpg" width="" height="125">
</picture>
  
<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_presto_hue/blob/main/images/ha_proxy.png" width="300" height="100">
</picture>
</p>

<br/><br/>

We are currently working on deploying a <b>3-NameNode Hadoop-Hive cluster</b>.
To support this setup, we need some additional tools — specifically, <b>Docker and HAProxy</b>.

This repository will be used to <b>install and configure both Docker and HAProxy</b> for the cluster environment.

The steps below show how to complete the setup.

# Docker

<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_hue_fast_access/blob/main/images/docker.jpg" width="" height="125">
</picture>

## Step 1: Install Docker on Ubuntu

The following steps will guide you through the Docker installation process on an Ubuntu environment.


### 1.1 Update package index and install prerequisites:

```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release
```

### 1.2 Add Docker's official GPG key:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### 1.3 Add Docker repository:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 1.4 Install Docker Engine:

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
sudo apt install docker-compose-plugin
sudo apt install docker-compose

sudo apt install python-is-python3 # For Presto
```

### Verify Installation

```bash
# Start and Enable service
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker

# Add current user to docker group
sudo usermod -aG docker $USER

# Check Docker version
docker --version

# Check Docker Compose version
docker compose version
# or if using standalone version:
docker-compose --version

# Test Docker installation
sudo docker run hello-world
```

<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_hue_fast_access/blob/main/images/docker_log.JPG" width="800" height="400">
</picture>


# HAProxy

<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_presto_hue/blob/main/images/ha_proxy.png" width="" height="125">
</picture>


## Step 1: Install HAproxy on Ubuntu

The following steps will guide you through the HAProxy installation process on an Ubuntu environment.


### 1.4 Install HAProxy Engine:

```bash
sudo apt update
sudo apt install haproxy
```

### Verify Installation

```bash
# Start and Enable service
sudo systemctl start haproxy
sudo systemctl enable haproxy
sudo systemctl status haproxy
```


### Configuration for Hadoop Cluster

```bash
# Edit HAproxy configuration file
sudo nano /etc/haproxy/haproxy.cfg

# Check the config file
sudo haproxy -f /etc/haproxy/haproxy.cfg -c

sudo systemctl restart haproxy
```

```xml
hadoop@slv05:~$ cat  /etc/haproxy/haproxy.cfg
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

# Web Interface Configurations
listen stats
    mode http
    bind *:8404
    stats enable
    stats uri /haproxy?stats
    stats refresh 10s
    stats show-legends
    stats show-node
    stats show-desc "Open Source Hadoop Cluster Statistics"
    stats show-modules
    stats hide-version
    stats auth hadoop:hadoop


# ===============================================
# MARIADB GALERA CLUSTER CONFIGURATIONS
# ===============================================


# MariaDB Write Requests
listen mariadb_write
    bind *:3307
    mode tcp
    option mysql-check user haproxy_check
    log global
    option tcplog
    server Mariadb_M01 mariadb01:3306 check
    server Mariadb_M02 mariadb02:3306 check backup
    server Mariadb_M03 mariadb03:3306 check backup

# MariaDB Read Requests (Load-balanced)
listen mariadb_read
    bind *:3306
    mode tcp
    option mysql-check user haproxy_check
    log global
    option tcplog
    balance roundrobin
    server Mariadb_M01 mariadb01:3306 check
    server Mariadb_M02 mariadb02:3306 check
    server Mariadb_M03 mariadb03:3306 check

```
<br/>
<br/>

Login to HAProxy portle by below link , username/password : hadoop/hadoop (You can configure URL and user/password in the config file)

http://IP:84443/haproxy?stats

<picture>
  <img alt="docker" src="https://github.com/kavindatk/haproxy_docker_setup/blob/main/images/haproxy.JPG" width="" height="500">
</picture>

<picture>
  <img alt="docker" src="https://github.com/kavindatk/haproxy_docker_setup/blob/main/images/haproxy2.JPG" width="" height="500">
</picture>
