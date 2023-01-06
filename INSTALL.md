# Deployment Guide

Please follow this guide to deploy HMS on a brand new server. This guide
implements a traditional server deployment using systemd and nginx.

[See here](https://github.com/nursingengineeringlab/hcm-k8s) for a microservice
style deployment, using kubernetes and haproxy.

## Deploying the Server

The server runs the following server software packages:

* hcm-frontend
* hcm-api
* hcm-datafetcher
* nginx
* mariadb-server
* emqx

### The Operating System

HMS server requires a Linux based operating system to deploy. The recommended
distribution and version is Ubuntu 20.04 LTS, however Ubuntu 22.04 will work as
well, with minor changes to the process.

### Installing system requirements


