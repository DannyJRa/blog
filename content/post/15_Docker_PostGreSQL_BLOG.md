---
title: Docker PostGreSQL 
author: DannyJRa
date: '2019-01-15'
slug: 15_dockerpostgresql 
categories:
  - Cloud
tags:
  - Docker
hidden: false
image: "img/15_Docker_PostGreSQL_BLOG.png"
share: false
output:
  html_document:
    keep_md: yes
    theme: cerulean
    highlight: tango
    code_folding: show
    toc: yes
    toc_float: yes
  pdf_document:
    number_sections: yes
geometry: margin = 1.2in
fontsize: 10pt
always_allow_html: yes

---

The reticulate package provides a comprehensive set of tools for interoperability between Python and R.

<!--more-->













# Docker: PostgreSQL 10 and pgAdmin 4

## Run Docker with PostgreSQL container

Make persistent volume to store config files and database files:

>mkdir -p $HOME/docker/volumes/postgres

Rund docker command to create postgresql database with User: postgres and POSTGRES_PASSWORD=docker:
# Exucute postgresql 



>docker run --rm   --name pg-docker -e POSTGRES_PASSWORD=docker -d -p 5434:5432 -v $HOME/docker/volumes/postgres:/var/lib/postgresql/data  postgres

## Connect to Postgres

Once the container is up an running, connecting to it from an application is no different than connecting to a Postgres instance running outside a docker container. For example, to connect using psql we can execute[^1]

>psql -h localhost -U postgres -d postgres

### Access postgresql.conf file

Switch to root 
>sudo su


And access the conf file to manage e.g. remote access to the database:

>/home/danny/docker/volumes/postgres# sudo nano postgresql.conf


### Open port

If you work on the google cloud, you can open the port=5434 for all instances in the current workind directory with:

>gcloud compute firewall-rules create postgresdocker --allow tcp:5434 \
      --description "Allow incoming traffic on TCP port 5434" \
      --direction INGRESS

      
##  Run Docker with pgAdmin 4 container

Pull and start the pgAdmin container:





```bash
docker run -p 5051:5051 \
        -e "PGADMIN_DEFAULT_EMAIL=${S_email}" \
        -e "PGADMIN_DEFAULT_PASSWORD=${S_pwd_pgadmin}" \
        -e "PGADMIN_LISTEN_PORT=5051" \
        -d dpage/pgadmin4
```

Problem: Not yet solved the issue that the volume will not be shared currently:


```bash
>docker run -p 5050:5050 \
        -e "PGADMIN_DEFAULT_EMAIL=xxxoutlook.com" \
        -e "PGADMIN_DEFAULT_PASSWORD=secretpassword" \
        -e "PGADMIN_LISTEN_PORT=5050" \
        -d dpage/pgadmin4 \
        -v "/home/danny/docker/volumes/pgadmin:/var/lib/pgadmin/shared"
```


### Open port 5050 on GCP


>gcloud compute firewall-rules create streamset --allow tcp:18630 \
      --description "Allow incoming traffic on TCP port 18630" \
      --direction INGRESS
 
        
List all firewall rules:

>gcloud compute firewall-rules list

Then login with your set credentials at port 5050 (in this case):
![Login](/img/pgAdmin_login.png) 
        
[^1]: Adopted from https://hackernoon.com/dont-install-postgres-docker-pull-postgres-bee20e200198

