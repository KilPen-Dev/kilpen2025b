---
title: "Migrate Existing Docker Peertube Instance to Linode, Enable Object Storage"
meta_title: "Peertube Migration Guide"
description: "Relocate a Peertube video hosting platform to Linode while implementing object storage"
date: 2023-07-25T00:00:00Z
image: "/images/solutions/migrate-peertube-to-linode.png"
categories: ["How To"]
author: "Chris"
tags: ["peertube", "docker", "linode", "migration", "object-storage"]
draft: false
---

<script type="text/javascript" src="https://cdnjs.buymeacoffee.com/1.0.0/button.prod.min.js" data-name="bmc-button" data-slug="kilpen" data-color="#FFDD00" data-emoji=""  data-font="Cookie" data-text="Buy me a coffee" data-outline-color="#000000" data-font-color="#000000" data-coffee-color="#ffffff" ></script>

SKILL LEVEL:  Advanced

{{< notice "info" >}}
This article assumes a familiarity of Linux administration, Docker, web apps, and hosting services. Some details may have been intentionally or unintentionally omitted.
{{< /notice >}}

## Overview
‌This project will be migrating an existing Peertube instance from one cloud service provider's Virtual Private Server (VPS) to a Linode VPS.  The existing Peertube instance, let's call that Peertube A, is an out-of-the-box install based on [Peertube's Docker install](https://docs.joinpeertube.org/install/docker) instructions.

**Peertube A** configurations consists of:
* SSH-enabled with security rules implemented
* Docker container using Docker Compose
* Nginx Webserver
* Peertube application
* Postgres Database
* Postfix relay to Google SMTP
* Redis
* Peertube storage and database within Docker on the VPS

The end-state configuration, let's call this Peertube B, will be slightly different.
**Peertube B** configurations will consist of:
* SSH-eable with security rules implemented
* Docker container user Docker Compose
* Nginx Webserver
* Peertube application
* Postgres Database
* Postfix relay to MailGun
* Redis
* Peertube storage using Linode Object Storage
* Pertube database within Docker on the VPS‌

## Create a Linode Virtual Private Server
On the Create page, you'll be able to select a few options.  I chose the following:
* Images:  Debian 11
* Region: Atlana, GA (us-southeast)
* Linode Plan Shared CPU - Linode 4GB Plan

Next, you'll need to create a Root password for your Debian instance.  Ensure this password is fairly long and that you save it somewhere secure.

You will also have an option to an an SSH key to your Linode account.  I recommend taking a moment to do this.  Creating and managing SSH keys is beyond the scope of this article, but there plenty of resources available online to assist.  You can start here to learn more.  The rest of this article will assume you're using certificates to access SSH.

Leave the rest of the options as default and click the *Create Linode* button.  Your VPS will start provisioning and eventually switch to a running status.
You'll then find the IP Address and SSH address.  Take note of these as you'll need them in the next step.

## Initial VPS Configuration ##
‌Congrats, your VPS is up and running.  Now it's time to make some adjustments for better security.

### Access the VPS ###
Next, let's start by remoting into the VPS using SSH.  From the command prompt or terminal, type: 
```bash
ssh root@[YOUR IP ADDRESS]
```

Enter the password the same password used when initially creating the Linode VPS.

Now that you're remoted into the new VPS, we need to tighten up the configuration for security.

### Update Linux and existing packages ###
Let's make sure the existing packages are all up-to-date before moving forward. 
```bash 
apt-get update && apt-get upgrade
```

### Set a hostname and timezone ###
To set a hostname, type 
```bash
hostnamectl set-hostname [YOUR HOST]
```

Before setting the timezone, you'll need to know the correct way to provide the time zone.  
For example, in Hawaii we use 'Pacific/Honolulu'.  To find display a list of options, type 
```bash
timedatectl list-timezones
```

To set a timezone, type 
```bash
timedatectl set-timezone '[YOUR TIMEZONE]'
```
### Create a new user ###
Now we need to create a user other than the root account.  We need to do this next step because later we will disable the ability to log in using root.
To create a new limited user, type `adduser [USERNAME]`

Next add the user to the sudo group:
```bash
usermod -aG sudo [USERNAME]
```

### SSH configuration ###
Now we want to limit access to the root account to login via SSH. Also, lets prevent username/password logins complete opting for certificates only.  Depending on risk, you may also decide to enable Google Authenticator for 2-factor authentication.

## Install Docker ##
*Reference: [Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/)*

Let's start by adding the Docker repositories so that we can use apt to install and maintain Docker.

### Set up the repository ###
1. Update the apt package index and install packages to allow apt to use a repository over HTTPS:
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```

2. Add Docker’s official GPG key:
```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

3. Use the following command to set up the repository:
```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Install Docker Engine ###
1. Update the apt package index:
```bash
sudo apt-get update
```
2. Install Docker Engine, containerd, and Docker Compose.
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
3. Verify that the Docker Engine installation is successful by running the hello-world image:
```bash
sudo docker run hello-world
```

### Post-installation steps ###
*Reference: [Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/)*

A few more things to do:
1. Create the docker group.
```bash
sudo groupadd docker
```
2. Add your user to the docker group.
```bash
sudo usermod -aG docker $USER
```

Log out and log back in so that your group membership is re-evaluated.

You can also run the following command to activate the changes to groups:
```bash
newgrp docker
```

Verify that you can run docker commands without sudo.
```bash
docker run hello-world
```

This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.

## Configure Peertube, Postgres, Redis, and Nginx ##
Reference: [Peertube: Docker Guide](https://docs.joinpeertube.org/install/docker)

### Install ###
*PeerTube does not support webserver host change. Keep in mind your domain name is definitive after your first PeerTube start.*

Get the latest Compose file:
```bash
curl https://raw.githubusercontent.com/chocobozzz/PeerTube/master/support/docker/production/docker-compose.yml > docker-compose.yml
```

Get the latest env_file:
```bash
curl https://raw.githubusercontent.com/Chocobozzz/PeerTube/master/support/docker/production/.env > .env
```

Tweak the docker-compose.yml file there according to your needs:
```bash
sudo nano docker-compose.yml
```

Here is the docker-compose.yml for our project.  Note that we did NOT use the CertBot/LetsEncrypt for this project.  Instead, we used Cloudflare Origin Certificates.  This required an edit the NGINX configuration file.

```bash
version: "3.3"
services:
  webserver:
    image: chocobozzz/peertube-webserver:latest
    env_file:
      - .env
    ports:
     - "80:80"
     - "443:443"
    volumes:
      - type: bind
        # Switch sources if you downloaded the whole repository
        #source: ../../nginx/peertube
        source: ./docker-volume/nginx/peertube
        target: /etc/nginx/conf.d/peertube.template
      - assets:/var/www/peertube/peertube-latest/client/dist:ro
      - ./docker-volume/data:/var/www/peertube/storage
      - /home/kilpen/cloudflare:/etc/cloudflare
    depends_on:
      - peertube
    restart: "always"

  peertube:
    image: chocobozzz/peertube:production-bullseye
    networks:
      default:
        ipv4_address: 172.18.0.42
    env_file:
      - .env
    ports:
     - "1935:1935" # Comment if you don't want to use the live feature
    volumes:
      - assets:/app/client/dist
      - ./docker-volume/data:/data
      - ./docker-volume/config:/config
    depends_on:
      - postgres
      - redis
      - postfix
    restart: "always"

  postgres:
    image: postgres:13-alpine
    env_file:
      - .env
    volumes:
      - ./docker-volume/db:/var/lib/postgresql/data
    restart: "always"

  redis:
    image: redis:6-alpine
    volumes:
      - ./docker-volume/redis:/data
    restart: "always"

  postfix:
    image: mwader/postfix-relay
    env_file:
      - .env
    volumes:
      - ./docker-volume/opendkim/keys:/etc/opendkim/keys
    restart: "always"

networks:
  default:
    ipam:
      driver: default
      config:
      - subnet: 172.18.0.0/16

volumes:
  assets:
  certbot-www:
```

Then tweak the .env file to change the environment variables settings
```bash
sudo nano .env
```

Here is the *.env* for our project.
```bash
# Database / Postgres service configuration
POSTGRES_USER=[YOUR DATABASE USER]
POSTGRES_PASSWORD=[YOUR DATABASE PASSWORD]
POSTGRES_DB=peertube

# Peertube variables
PEERTUBE_DB_USERNAME=[YOUR DATABASE USER]
PEERTUBE_DB_PASSWORD=[YOUR DATABASE PASSWORD]
PEERTUBE_DB_SSL=false
PEERTUBE_DB_HOSTNAME=postgres

# PeerTube server configuration
PEERTUBE_WEBSERVER_HOSTNAME=[YOUR HOST AND DOMAIN]
PEERTUBE_SECRET=[GENERATE THIS]

PEERTUBE_SMTP_USERNAME=[INSERT MAILGUN USERNAME HERE]
PEERTUBE_SMTP_PASSWORD=[INSERT MAILGUN PASSWORD HERE]
PEERTUBE_SMTP_HOSTNAME=smtp.mailgun.org
PEERTUBE_SMTP_PORT=587
PEERTUBE_SMTP_FROM=postmaster@[YOUR DOMAIN]
PEERTUBE_SMTP_TLS=false
PEERTUBE_SMTP_DISABLE_STARTTLS=false
PEERTUBE_ADMIN_EMAIL=admin@[YOUR DOMAIN]

# Postfix service configuration
POSTFIX_myhostname=[YOUR DOMAIN]

# OBJECT STORAGE
PEERTUBE_OBJECT_STORAGE_ENABLED:true
PEERTUBE_OBJECT_STORAGE_ENDPOINT:[YOUR STORAGE ENDPOINT]
PEERTUBE_OBJECT_STORAGE_REGION:[YOUR REGION]
PEERTUBE_OBJECT_STORAGE_CREDENTIALS_ACCESS_KEY_ID:[YOUR BUCKET ACCESS KEY ID]
PEERTUBE_OBJECT_STORAGE_CREDENTIALS_SECRET_ACCESS_KEY:[YOUR BUCKET SECRET]

PEERTUBE_OBJECT_STORAGE_VIDEOS_BUCKET_NAME:peertube-prod
PEERTUBE_OBJECT_STORAGE_VIDEOS_PREFIX:videos/
PEERTUBE_OBJECT_STORAGE_STREAMING_PLAYLISTS_BUCKET_NAME:peertube-prod
PEERTUBE_OBJECT_STORAGE_STREAMING_PLAYLISTS_PREFIX:streaming-playlists/
```

## Migrate storage and database to Linode ##
We'll need to generate a new SSH keypair for this next step.  Using your tool of choice, generate a new public key and private key.  On Peertube A, place the private key in the home directory.  Remove read permissions:
```bash
chmod go-rw private.key
```

On Peertube B, update the *~/.ssh/authorized_keys* to include the key pair's public key.
### Migrate the data folder ###
Now run the following command to copy the data folder from Peertube A to Peertube B.
```bash
rsync -avz ~/docker-volume/data -e "ssh -i private.key" [YOUR USERNAME]@[YOUR IP ADDRESS]:~/peertube/docker-volume/`
```

### Migrate the db folder ###
Now run the following command to copy the db folder from Peertube A to Peertube B.  Since the file isn't owned
```bash
sudo rsync -avz ~/docker-volume/db -e "ssh -i private.key" [YOU USERNAME]@[YOUR IP]:~/peertube/docker-volume/
```

### Correct file ownership ###
The rsync command will preserve file permissions, but ownership need to be corrected.

On Peertube B, run following:
```bash
sudo chown 70:root ~/peertube/docker-compose/db`
```

## Move existing video files to object storage ##
Peertube already has a script to help with [moving videos to storage](https://docs.joinpeertube.org/maintain/tools#create-move-video-storage-job-js)

For this project, we will run the script from Peertube B within the *~/peertube/ *folder:
```bash
docker compose exec -u peertube peertube npm run create-move-video-storage-job -- --to-object-storage --all-videos
```

## Well Done! ##
Hopefully this article helped better understand the nuance details of migrating a Peertube server.

KilPen Technical Services is here to help with YOUR project.  [Contact us for assistance](https://www.kilpen.com/contact.html).


