# Docker backend stack

The docker-compose file contains basic stack to connect to when dealing with apps.

![alt text](https://i.ibb.co/FK2CK4f/docker-network.png)


### Install docker and docker compose

Refer to official doc :
 - docker : https://docs.docker.com/install/linux/docker-ce/ubuntu/
 - docker-compose : https://docs.docker.com/compose/install/
 
## Register github package

- Create a personal access token as described in github documentation : https://docs.github.com/en/packages/guides/pushing-and-pulling-docker-images#authenticating-to-github-container-registry
- Then apply step 2 & 3 in order to sign in to the GitHub Container Registry service.   

## Getting started

First, the shopping-feed [docker network](https://docs.docker.com/network/) must be created to make it available to other docker instances.
In order to avoid local prefixes, lets create it manually

```bash
docker network create --driver bridge --subnet=170.10.0.0/16 --ip-range=170.10.5.0/24 sf_network
```

Note that every containers in this network declare or acquire an ipv4 address from 170.10.5.0 to 170.10.5.255.

By convention, ip ranges are reserved as following :

- 170.10.5.0 -> 170.10.5.99 : infrastructure services (databases, load-balancer... etc)
- 170.10.5.100 -> 170.10.5.199 : shoppingfeed applications

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
By doing this, all containers as a part of the network became available for your project.

If the container has to be accessed from another one (ie: api application, used by legacy app), you must specify

- container_name : By convention, declared with vendor prefix "sfeed.". Example : `sfeed.api.web`
- networks.sf_network.ipv4_address : Service static IP, see network chapter above for ip conventions.

Example of basic declaration for the API project found in [docker-compose.yml file](https://github.com/shoppingflux/api/blob/master/docker-compose.yml)

```yaml
version: "3"
services:
  app_api:  
    # Makes the container available to that domain name
    container_name: sfeed.api.web
    image: "shoppingfeed:api"
    volumes:
    - "$PWD:/var/www"
    # This will add the container and allow other containers access
    networks:
      sf_network:
        ipv4_address: "170.10.5.100"

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
- network ip: 170.10.5.15
- container name: sfeed.traefik

#### Configuration

If your application or service must be exposed to http protocol and should be considered as a backend,
you can configure it like following in the docker-compose file (the example use the already configured elements for the API application)

```yaml
version: "3"
services:
  app:
    container_name: sfeed.api.web
    image: "shoppingfeed:api"
    networks:
      sf_network:
        - ipv4_address: "170.10.5.100"
    labels:
      # Enable traefik to create front and backend
      traefik.enable: "true"
      # define a hostname for your frontend service
      traefik.frontend.rule: "Host:api.shopping-feed.lan"
      # Optionally define the backend name
      traefik.backend: "sfeed.api.web"
      # define the network that the backend belongs to
      traefik.docker.network: "sf_network"
      # If you want only expose the service for http (or https)
      traefik.frontend.entryPoints: "http"

networks:
  sf_network:
    external: true
```

This will make your app accessible at http://api.shoppingfeed.lan on host machine.

To get more information about how to use traefik, check https://docs.traefik.io/basics/

You can also check available options for docker-compose integration here : https://docs.traefik.io/configuration/backends/docker/#using-docker-compose

An API microservice configuration example with billing application and url path matching here : https://github.com/shoppingflux/app-billing/blob/master/docker-compose.yml#L17

### RabbitMQ

#### Docker network

- ip: 170.10.5.12
- host: sfeed.rabbitmq
- port: 5672
- user: admin
- pass: root

#### Host machine

- port forwarding: `8001:5672`
- web admin: `http://rbmq.shopping-feed.lan`

### Redis

#### Docker network

- ip: 170.10.5.13
- host: sfeed.redis
- port: 6379
- user: N/A
- pass: N/A

#### Host machine

- port forwarding: `8002:6379`

### ElasticSearch

#### Docker network

- ip: 170.10.5.11
- host: sfeed.elasticsearch
- port: 9200
- user: N/A
- pass: N/A

#### Host machine

- port forwarding: `8004: 9200`
- http access: `http://es.shopping-feed.lan`

### Kibana

#### Docker network

- ip: 170.10.5.10
- host: sfeed.kibana
- port: 5601
- user: N/A
- pass: N/A

#### Host machine

- port forwarding: `8006:5601`
- http access: `http://kibana.shopping-feed.lan`

### MongoDB

#### Docker network

- ip: 170.10.5.14
- host: sfeed.mongodb
- port: 27017
- user: N/A
- pass: N/A

#### Host machine

- port forwarding: `8003:27017`

### MariaDB

#### Docker network

- ip: 170.10.5.16
- host: sfeed.mariadb
- port: 3306
- user: root
- pass: root
- defaut db: shopping_feed

#### Host machine

- port forwarding: `8005:3306`

### S3 server



#### Docker network

- ip: 170.10.5.17
- host: sfeed.s3
- port: 9000
- user: admin
- pass: password

#### Host machine

- port forwarding: `8007:9000`
- http access (browser supported): `http://s3.shopping-feed.lan`
- files are stored in `./data/s3/*` folders (* are buckets names)
