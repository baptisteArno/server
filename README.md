My dead simple setup for self-hosting applications with docker-compose (and traefik).

This config self-sign new applications automatically and make them available to the URL of your choice.

## Prerequisite

- A server with a working installation of docker and docker-compose
- A domain with the following DNS records:
```
Type    Name     Value
A       @        <VPS_IP>
```
This record tells that the URL `example.com` should point to the VPS IP.
```
Type    Name     Value
A       *        <VPS_IP>
```
This record tells that the URLs `*.example.com` should point to the VPS IP.

## Install

1. Clone the repository
2. Create an `acme.json` file
3. Copy all the `.env.example` files to `.env`
4. Fill the `.env` files with your config
5. Run `docker compose up`


## How to

### Add a new application

1. Create and configure a new compose service `apps/newapp/newapp.yml`

```yml
version: "3.3"

services:
  newapp:
    image: <DOCKER_IMAGE>
    container_name: "newapp"
    labels:
      - "traefik.enable=true"
      - "traefik.http.services.newapp.loadBalancer.server.port=<EXPOSED_PORT_BY_CONTAINER>"
      - "traefik.http.routers.newapp.rule=Host(`newapp.${BASE_DOMAIN}`)"
      - "traefik.http.routers.newapp.entrypoints=websecure"
      - "traefik.http.routers.newapp.tls.certresolver=myresolver"
```

Replace <DOCKER_IMAGE> and <EXPOSED_PORT_BY_CONTAINER>.

2. List the application in the root `docker-compose.yml`:

```yml
newapp:
  extends:
    file: apps/newapp/newapp.yml
    service: newapp
```

3. It should create an app available at https://newapp.example.com 🚀

### Upgrade an app

```sh
docker compose pull app
docker compose up -d --force-recreate --no-deps --build app
```