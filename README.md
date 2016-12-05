goldy/syncthing
===================

Container based on https://github.com/istepanov/docker-syncthing

[![Docker Stars](https://img.shields.io/docker/stars/goldy/syncthing.svg)](https://hub.docker.com/r/goldy/syncthing/)
[![Docker Pulls](https://img.shields.io/docker/pulls/goldy/syncthing.svg)](https://hub.docker.com/r/goldy/syncthing/)
[![Docker Build](https://img.shields.io/docker/automated/goldy/syncthing.svg)](https://hub.docker.com/r/goldy/syncthing/)
[![Layers](https://images.microbadger.com/badges/image/goldy/syncthing.svg)](https://microbadger.com/images/goldy/syncthing)
[![Version](https://images.microbadger.com/badges/version/goldy/syncthing.svg)](https://microbadger.com/images/goldy/syncthing)


[Syncthing](http://syncthing.net/) Docker image

### How to run

Simpliest way to run this container is by using [docker-compose.yml](docker-compose.yml) file. Simply run:

    wget https://raw.githubusercontent.com/istepanov/docker-syncthing/master/docker-compose.yml
    docker-compose up -d

Then access Syncthing Web UI at [http://localhost:8384/]().

To update the container, run:

    docker-compose pull
    docker-compose up -d

You can also start the container manually without using Docker Compose:

    # Container requires 2 volumes: for config and for data.
    CONFIG_VOLUME='syncthing-config'
    DATA_VOLUME='syncthing-data'
    CONTAINER_NAME='syncthing'

    docker run -d \
        --name $CONTAINER_NAME \
        --restart always \
        -p 8384:8384 -p 22000:22000 -p 21027:21027/udp \
        -v $CONFIG_VOLUME:/home/syncthing/.config/syncthing \
        -v $DATA_VOLUME:/home/syncthing/Sync \
        goldy/syncthing

_Note_: `--restart always` (or `--restart on-failure` or `--restart unless-stopped`) is required because Syncthing restarts itself during auto-update. Without this option container just stops after first update.

#### Change volume `UID` and `GID`

By default, `UID` and `GID` for the files in sync volume are 1000. You can change this by passing `ENTRYPOINT_USER` and `ENTRYPOINT_GROUP` environment variable to the containers.

    -e ENTRYPOINT_USER=33 -e ENTRYPOINT_GROUP=33

or using `docker-compose.yml` (see example).

```yaml
    environment:
        ENTRYPOINT_USER: '33'
        ENTRYPOINT_GROUP: '33'
```    

### Enabling HTTPS for Web UI

The image itself tends to be minimal and doesn't provide any HTTPS functionality. If you want to have a SSL connection between your browser and Syncthing Web UI, you need to put a proxy container that will handle HTTPS connections.

The easiest way to do it is by using Let's Encrypt certifcates and 3 additional containers: [official Nginx image](https://hub.docker.com/_/nginx/), [jwilder/docker-gen](https://github.com/jwilder/docker-gen) image to auto-generate Nginx configs and [jrcs/letsencrypt-nginx-proxy-companion](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion) image to auto-generate/auto-renew SSL certificates. To orchestrate all containers, we'll use [docker-compose-https.yml](docker-compose-https.yml) file:

    wget https://raw.githubusercontent.com/istepanov/docker-syncthing/master/docker-compose-https.yml

    # before launching, we need to prepare 2 docker volumes:
    docker volume create --driver local --name certs
    docker volume create --driver local --name nginx-templates

    # download nginx template into "nginx-templates" volume
    docker run --rm -v nginx-templates:/tmp/nginx-templates alpine /bin/sh -c 'apk add --no-cache curl ca-certificates && curl -o /tmp/nginx-templates/nginx.tmpl https://gist.githubusercontent.com/istepanov/6c19d1e519196e9b74f76344353fe837/raw/fc8784d1cbc8ad56047b10630b68c1830859bf63/nginx.tmpl'

    # Setup env vars that will be used by Docker Compose file.
    # SYNCTHING_HOSTNAME must be accessible from outside, otherwise SSL certificate won't be generated.
    # Make sure port 80 is open and the domain name is resolvable.
    # (this is an example, replace this with real values)
    export SYNCTHING_HOSTNAME='syncthing.myserver.host'
    export LETSENCRYPT_EMAIL='my@email.host'

    docker-compose -f docker-compose-https.yml up -d

Make sure you set `LETSENCRYPT_EMAIL` and `LETSENCRYPT_EMAIL` env vars before running Docker Compose commands. You can also edit `docker-compose-https.yml` and replace these vars with real values.
