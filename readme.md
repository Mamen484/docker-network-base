# Docker backend stack

The docker-compose file contains basic stack to connect to when dealing with apps.

## Getting started

First, the shopping-feed network must be created.
In order to avoid local prefix, lets create it manually

```bash
docker network create --driver bridge sf_network
```

Once done, you can start the stack

```bash
docker-compose up -d
```

Network can be inspected with the following command

```bash
docker network inspect sf_network
```

## Connect to the network

In order to connect your app to the network, you must record `sf_network` and explicitly link your containers to it

Example of basic declaration for the API app

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

Traefik is the http router installed into the stack and is configured to expose http services on the host port `80`

- http applications: http://{domain}:80
- http dashboard: http://localhost:8080

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
      # define a hostname for your frontend service
      traefik.frontend.rule: "Host:api.shopping-feed.lan"
      # define the backend name`
      traefik.backend: "api"
      # define the network that the backend belongs to
      traefik.docker.network: "sf_network"

networks:
  sf_network:
    external: true
```

This will make your app accessible at http://api.shoppingfeed.lan on host machine

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



 
