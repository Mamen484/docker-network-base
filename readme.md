# Docker backend stack

The docker-compose file contains basic stack to connect to when dealing with apps.

![alt text](https://i.ibb.co/FK2CK4f/docker-network.png)

## Prerequisit

First you need to login into AWS to be able to retrieve sf docker images.

### Get access to AWS Container Registry (ECR)

1. Create AWS Credentials and Config file

First install AWS Cli :

On a OSx machine:
```bash
$ brew install awscli   # install aws cli
``````
   
On a Linux machine:
```bash
$ pip install awscli --upgrade --user   # install aws cli
``````
Then configure credentials

```bash
$ aws configure
1- Enter AWS Access Key ID 
2- Enter AWS Secret Access Key
3- Region: eu-west-1
4- output: json

It wil generate those 2 files in ~/.aws:
# ~/.aws/credentials
[default]
aws_access_key_id=<id>
aws_secret_access_key=<access_key>

# ~/.aws/config
[default]
region=eu-west-1
output=json
```

2. Login

```
$ $(aws ecr get-login --no-include-email)  # log into ECR
```

### Install docker and docker compose

Refer to official doc :
 - docker : https://docs.docker.com/install/linux/docker-ce/ubuntu/
 - docker-compose : https://docs.docker.com/compose/install/

## Getting started

First, the shopping-feed [docker network](https://docs.docker.com/network/) must be created to make it available to other docker instances.
In order to avoid local prefix, lets create it manually

```bash
docker network create --driver bridge sf_network
```

Once done, you can start the entire stack

```bash
docker-compose up -d
```

Note that you can selective boot some of services, like that

```bash
docker-compose up -d mariadb redis
```

At any moment, if you need a new service, boot it, it will join the network automatically and connect to already running services

```bash
docker-compose up -d mongodb
```

Network can be inspected with the following command

```bash
docker network inspect sf_network
```

### Troubleshoot

If you have error about binaries not found you might need to add `~/.local/bin` to your `$PATH` variable. 

If you have issues while sending request to an external API (request hanging and finally time out) you must create the network with a lower mtu value :

```bash
docker network create --opt com.docker.network.driver.mtu=1400 --driver bridge sf_network
```


## Connect to the network

In order to connect your project to the network, you must record `sf_network` and explicitly link your containers to it.
By doing this, all containers as a part of the network became available for your project

Example of basic declaration for the API project found in [docker-compose.yml file](https://github.com/shoppingflux/api/blob/master/docker-compose.yml)

```yaml
version: "3"
services:
  app_api:
    image: "shoppingfeed:api"
    volumes:
    - "$PWD:/var/www"
    # This will add the container and allow other containers access
    networks:
      - sf_network

# Declare pre-existing network
networks:
  sf_network:
    external: true
```

## Available services

The stack by default connect itself to the `sf_network`

### Traefik

Traefik is the http router installed into the stack and is configured to expose http services on both host ports `80` and `443` by default.

- http applications: http://{domain}:80
- https applications: https://{domain}:443
- http dashboard: http://localhost:8080

#### Use https

Https must 


#### Configuration

If your application or service must be exposed to http protocol and should be considered as a backend,
you can configure it like following in the docker-compose file (the example use the already configured elements for the API application)

```yaml
version: "3"
services:
  app:
    image: "shoppingfeed:api"
    networks:
      - sf_network
    labels:
      # Enable traefik to create front and backend
      traefik.enable: "true"
      # define a hostname for your frontend service
      traefik.frontend.rule: "Host:api.shopping-feed.lan"
      # Optionally define the backend name
      traefik.backend: "api"
      # define the network that the backend belongs to
      traefik.docker.network: "sf_network"
      # If you want only expose the service for http (or https)
      traefik.frontend.entryPoints: "http"

networks:
  sf_network:
    external: true
```

This will make your app accessible at http://api.shoppingfeed.lan on host machine.

To get more informations about how to use traefik, check https://docs.traefik.io/basics/

You can also check available options for docker-compose integration here : https://docs.traefik.io/configuration/backends/docker/#using-docker-compose

An API microservice configuration example with billing application and url path matching here : https://github.com/shoppingflux/app-billing/blob/master/docker-compose.yml#L17

### RabbitMQ

#### Docker network

- host: rabbitmq
- port: 5672
- user: admin
- pass: root

#### Host machine

- port forwarding: `8001:5672`
- web admin: `http://rbmq.shopping-feed.lan`

### Redis

#### Docker network

- host: redis
- port: 6379
- user: N/A
- pass: N/A

#### Host machine

- port forwarding: `8002:6379`

### ElasticSearch

#### Docker network

- host: elasticseach
- port: 9200
- user: N/A
- pass: N/A

#### Host machine

- port forwarding: `8004: 9200`
- http access: `http://es.shopping-feed.lan`

### Kibana

#### Docker network

- host: kibana
- port: 5601
- user: N/A
- pass: N/A

#### Host machine

- port forwarding: `8006:5601`
- http access: `http://kibana.shopping-feed.lan`

### MongoDB

#### Docker network

- host: mongo
- port: 27017
- user: N/A
- pass: N/A

#### Host machine

- port forwarding: `8003:27017`

### MariaDB

#### Docker network

- host: mariadb
- port: 3306
- user: root
- pass: root
- defaut db: shopping_feed

#### Host machine

- port forwarding: `8005:3306`



 
