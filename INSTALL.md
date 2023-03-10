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

The Web frontend needs to access the EMQX server through reverse proxied web
sockets. This is achieved by copying the contents of
[this file](etc/emqx.nginx.conf) into the appropriate section of the nginx
configuration file `/etc/nginx/sites-available/default`. Keep in mind nginx
configuration file changes only takes effect after a service restart.

### Redis Time Series

The default Redis installation shipped with Ubuntu does not come with Redis Time
Series module. We need to introduce the module manually.

This repository contains a pre-built Redis Time Series module for Linux amd64
platform. We can simply use that module:

```bash
echo loadmodule /opt/hcm-docs/lib/redistimeseries.so | \
	sudo tee -a /etc/redis/redis.conf
sudo systemctl restart redis
```

### MariaDB

We prepare the MariaDB database with a user and a database for the app:

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

### Install the HCM software packages

Each of the HCM software package has its individual installation guide. Follow
the instructions in `INSTALL.md` files of each component to install them.

### Starting the Server

Due to a limitation in Snap, EMQX installed through Snaps will not automatically start upon system boot. You have to start the service manually every time when
you restarted the server, along with the HCM Datafetcher that it depends on:

```bash
sudo emqx start
sudo systemctl restart hcm-datafetcher
```

## Deploying the Raspberry Pi nodes

The following steps descrive how the Raspberry Pi nodes are deployed.

### The Operating System

The Raspberry Pi nodes uses the official Raspberry Pi OS image as the basis. If
the node is based on Raspberry Pi 3, Raspberry Pi 4 or Raspberry Pi Zero 2, the
64-bit Raspberry Pi OS is preferred. Raspberry Pi Zero W requires the 32-bit
version. We recommend using the Lite version (headless server version) to reduce
system resource and power consumption, as well as potential attack surface.

The latest release Raspberry Pi OS no longer have a default user and password,
and we need SSH access to the node if we are running it headless. In order to prepare for that, we do the following to the boot partition after the image is
written to the SD card but before it is transferred to the Raspberry Pi:

```bash
cd /media/`whoami`/boot # change this to reflect where the boot partition is.
touch ssh
# change the user name and password as necesssary.
echo username:`echo 'mypassword' | openssl passwd -6 -stdin` > userconf.txt
```

### Installing system requirements

We use the following command to install all third-party packages our project
depends on:

```bash
sudo apt update
sudo apt install python3 python-venv libglib2.0-dev
```

### Checkout all the code

We put all the code into `/opt` folder.

```bash
sudo mkdir /opt
sudo chown `id -un`:`id -gn` /opt
cd /opt
git clone https://github.com/nursingengineeringlab/pyclient.git
```

### Install the HCM software packages

Each of the HCM software package has its individual installation guide. Follow
the instructions in `INSTALL.md` files of each component to install them.

### Running the Raspberry Pi code

```bash
sudo screen /opt/pyclient/venv/bin/python3 /opt/pyclient/ble.py
```

## Copyright

Copyright &copy; 2021-2023 University of Massachusetts Amherst
