# Docker and HAProxy Setup for 3-NameNode Hadoop-Hive Cluster 

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

