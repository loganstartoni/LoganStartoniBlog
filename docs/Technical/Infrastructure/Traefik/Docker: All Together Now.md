---
title: Traefik - All Configured
summary: The Full Configuration for traefik setup in docker.
authors:
    - Logan Startoni
date: 2023-09-10
---
# Overview
This is the full configuration for the traefik web proxy. This Configuration is being used to proxy services run in the local environment as well as being remoted into a NATed network without any firewall holes. This is done by linking the machine to a machine in the DMZ via tailscale. 

# Sanitized Configuration File
```yaml
version: '3.5'

services:
  reverse-proxy:
    # The official v2 Traefik docker image
    image: traefik:latest
    restart: always
    container_name: reverseproxy
    hostname: reverseproxy
    environment:
      - "NAMECHEAP_API_USER=${NAMECHEAP_API_USER}"
      - "NAMECHEAP_API_KEY=${NAMECHEAP_API_KEY}"
      - "PUID=1000"
      - "PGID=1000"
    command:
      # Logging Flags
      - "--log.level=DEBUG"
      # - "--log.level=INFO"
      - "--log.filepath=/traefiklog/traefik.log"
      - "--accesslog"
      # Admin Flags
      - "--api"
      # HTTP
      - "--entryPoints.web.address=:80"
      # HTTPS
      - "--entrypoints.websecure.address=:443"
      # HTTPS - Create Certificates
      - "--certificatesresolvers.le.acme.tlschallenge=true"
      - "--certificatesresolvers.le.acme.email=${DOCKER_TRAEFIK_LETSENCRYPT_EMAIL}"
      - "--certificatesresolvers.le.acme.storage=/letsencrypt/acme.json"
      - "--certificatesresolvers.le.acme.tlschallenge=true"
      # HTTPS - Create Certificates - dev
      - "--certificatesresolvers.leDev.acme.tlschallenge=true"
      - "--certificatesresolvers.leDev.acme.email=${DOCKER_TRAEFIK_LETSENCRYPT_EMAIL}"
      - "--certificatesresolvers.leDev.acme.storage=/letsencrypt/acme_dev.json"
      - "--certificatesresolvers.leDev.acme.dnschallenge.provider=namecheap"
      - "--certificatesresolvers.leDev.acme.dnschallenge.delaybeforecheck=30"
      - "--certificatesresolvers.leDev.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.leDev.acme.tlschallenge=true"
      # HTTPS - Create Certificates - dns
      - "--certificatesresolvers.ledns.acme.tlschallenge=true"
      - "--certificatesresolvers.ledns.acme.email=${DOCKER_TRAEFIK_LETSENCRYPT_EMAIL}"
      - "--certificatesresolvers.ledns.acme.storage=/letsencrypt/acme_dns.json"
      - "--certificatesresolvers.ledns.acme.dnschallenge.provider=namecheap"
      - "--certificatesresolvers.ledns.acme.dnschallenge.delaybeforecheck=30"
      - "--certificatesresolvers.ledns.acme.tlschallenge=false"
      # HTTPS - Create Certificates - dns - dev
      - "--certificatesresolvers.lednsDev.acme.tlschallenge=true"
      - "--certificatesresolvers.lednsDev.acme.email=${DOCKER_TRAEFIK_LETSENCRYPT_EMAIL}"
      - "--certificatesresolvers.lednsDev.acme.storage=/letsencrypt/acme_dev_dns.json"
      - "--certificatesresolvers.lednsDev.acme.dnschallenge.provider=namecheap"
      - "--certificatesresolvers.lednsDev.acme.dnschallenge.delaybeforecheck=30"
      - "--certificatesresolvers.lednsDev.acme.tlschallenge=false"
      ## For Dev Only
      - "--certificatesresolvers.leDev.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"
      - "--certificatesresolvers.lednsDev.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"

      # Providers
      ## Docker Provider
      - "--providers.docker"
      - "--providers.docker.exposedbydefault=false"
      ## File Provider for Custom local services
      - "--providers.file=true"
      - "--providers.file.directory=/traefikconfig/routes"
      - "--providers.file.watch=true"
    labels:
      # Global redirect to HTTPS
      - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)"
      - "traefik.http.routers.http-catchall.entrypoints=web"
      - "traefik.http.routers.http-catchall.middlewares=redirect-to-https"
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"

      # Dashboard over HTTPS
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.rule=Host(`dash.${TRAEFIK_DOMAIN}`)"
      - "traefik.http.routers.traefik.service=api@internal"
      - "traefik.http.routers.traefik.tls.certresolver=le${DOCKER_CERT_RESOLVER_SUFFIX}"
      - "traefik.http.routers.traefik.entrypoints=websecure"
      - "traefik.http.middlewares.ipwhitelist.ipwhitelist.sourcerange=${DOCKER_TRAEFIK_WHITELIST}"
      - "traefik.tcp.middlewares.ipwhitelist.ipwhitelist.sourcerange=${DOCKER_TRAEFIK_WHITELIST}"
      - "traefik.http.routers.traefik.middlewares=ipwhitelist"
    ports:
      - "80:80"
      - "443:443"
      # The Web UI (enabled by --api.insecure=true)
      - "8080:8080"
    volumes:
      # Allows Traefik to listen to the Docker events
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      # Allows Certificate Storage across Sessions
      - "${DOCKER_HOME}/traefik/letsencrypt:/letsencrypt"
      # Used in the file Provider
      - "${DOCKER_TRAEFIK_FILE_PROVIDER}:/traefikconfig/routes"
      # Log File
      - "${DOCKER_LOGS}/traefik/log:/traefiklog"
      # ???
      - "${DOCKER_HOME}/traefik/data:/traefik"
    networks:
      - traefik
      - traefik-proxy

networks:
  traefik:
    name: traefik
    external: true
  traefik-proxy:
    name: traefik-proxy
    external: false

```

# Environment Variables to be configured
- DOCKER_TRAEFIK_DOMAIN
    - The domain name that will be used. This should have a wildcard domain entry in your dns provider. This will allow you to add additional configuration without updating your domain.
- DOCKER_HOME
    - Where all of your docker volumes are located. 
- DOCKER_TRAEFIK_FILE_PROVIDER
    - Where the files are located for the traefik file provider
- DOCKER_LOGS
    - Where the docker logs should be saved. 
- DOCKER_CERT_RESOLVER_SUFFIX
    - A quick way to change whether we are using the dev or prod version of the certificate resolvers
    - This should be set to either 'Dev' or not set at all. 
- DOCKER_TRAEFIK_LETSENCRYPT_EMAIL
    - The email that will be used to register with letsencrypt. 
- NAMECHEAP_API_USER
    - The namecheap user to be used when using dns verification. 
    - If you are not using namecheap you will need to update this to match your dns provider.
- NAMECHEAP_API_KEY
    - The namecheap api key.
    - If you are not using namecheap you will need to update this to match your dns provider.
- DOCKER_TRAEFIK_WHITELIST
    - The whitelist that will be used to only allow some IPs to access the services assigned this middleware.