# Docker backend stack

The docker-compose file contains basic stack to connect to when dealing with apps.

### Getting started

First, the shopping-feed network must be created.
In order to avoid local prefix, lets create it manually

```bash
docker network create --driver bridge sf_network
```

Once done, you can start the stack

```bash
docker-compose up -d
```


### In details

The stack by default connect itself to the `sf_network`, and contains

syntax : name(host port,network port)

- traefik (80:80, 8080:8080)
- rabbitmq (8001:5672)
- redis (8002:6379)
- elasticsearch (8004:9200)
- mongodb (8003:27017)


### Connect to the network

In order to connect your app to the network, you must record `sf_network` and explicitly link your containers to it

Example of basic declaration for the API app

```yaml
version: "3"
services:
  app_api:
    image: "813931216550.dkr.ecr.eu-west-1.amazonaws.com/api"
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

### Connect to traefik

Traefik is configured to expose http services on the port `80`


If your application or service must be exposed to http protocol and should be considered as a backend, you can configure it like following in the docker-compose file (the example use the already configured elements for the API application)

```yaml
version: "3"
services:
  app_api:
    image: "813931216550.dkr.ecr.eu-west-1.amazonaws.com/api"
    volumes:
    - "$PWD:/var/www"
    networks:
      - sf_network
    labels:
      # define a hostname for your frontend service
      traefik.frontend.rule: "Host:api.shopping-feed.lan"
      # define the backend name
      traefik.backend: "api"
      # define the network that the backend belongs to
      traefik.docker.network: "sf_network"

networks:
  sf_network:
    external: true
```

### Inspect networks

#### Using traefik

Open `http://localhost:8080` in your browser

#### Using docker

Type in terminal

```bash
docker network inspect sf_network
```

 
