# Deployment Guide

Please follow this guide to deploy HMS on a brand new server. This guide
implements a traditional server deployment using systemd and nginx, as well as
using MariaDB as the database server.

[See here](https://github.com/nursingengineeringlab/hcm-k8s) for a microservice
style deployment, using kubernetes and haproxy.

## Deploying the Server

The following steps describes how a server is deployed.

### The Operating System

HMS server requires a Linux based operating system to deploy. The recommended
distribution and version is Ubuntu 20.04 LTS, however Ubuntu 22.04 will work as
well, with minor changes to the process.

### Installing system requirements

We use the following command to install all third-party packages our project
depends on:

```bash
sudo apt update
sudo apt install python3 python-venv libglib2.0-dev nginx mariadb-server redis \
    git build-essential
```

Then we prepare the MariaDB database with a user and a database for the app:

```bash
sudo mysql_secure_installation
sudo mysql
```

```sql
CREATE DATABASE hcm;
GRANT ALL ON hcm.* TO hcm@localhost IDENTIFIED BY "hcm";
FLUSH PRIVILEGES;
EXIT;
```

### Checkout all the code

We put all the code into `/opt` folder.

```bash
sudo mkdir /opt
sudo chown `id -un`:`id -gn` /opt
cd /opt
for repo in hcm-api hcm-frontend hcm-datafetcher hcm-docs; do
	git clone https://github.com/nursingengineeringlab/$repo.git
done
```

### Install EMQX

EMQX installation differs between Ubuntu 20.04 and Ubuntu 22.04. For Ubuntu
20.04, [follow the official guide](https://www.emqx.io/downloads?os=Ubuntu). For
Ubuntu 22.04, install it from Snaps:

```bash
sudo snap install emqx
```

**NOTE:** EMQX installed through Snaps will not automatically start upon system
boot. You have to start the service manually every time when you restarted the
server, along with the HCM Datafetcher that it depends on:

```bash
sudo emqx start
sudo systemctl restart hcm-datafetcher
```

### Redis Time Series

The default Redis installation shipped with Ubuntu does not come with Redis Time
Series module. We need to introduce the module manually.
