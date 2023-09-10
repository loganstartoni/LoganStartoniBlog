---
title: Traefik - Command Breakdown
summary: The Traefik Command Configuration.
authors:
    - Logan Startoni
date: 2023-09-10
---
# Overview
This page will break down the parts of the Command in the  [docker file](Docker%3A%20All%20Together%20Now.md). This will allow you to extend it if you have to. 

Configuring the [command](Docker%3A%20Command%20Breakdown.md) to do what I wanted and the [labels](Docker%3A%20Labels%20Breakdown.md) is something that when I first dived into setting up my setup I struggled to understand. 

# Command Breakdown
## Logging Setup
```yaml
    command:
      # Logging Flags
      # - "--log.level=DEBUG"
      - "--log.level=INFO"
      - "--log.filepath=/traefiklog/traefik.log"
      - "--accesslog"
```
- `--log.level` 
    - sets the log level
- `--log.filepath`
    - Sets the filepath of where to save the traefik logs
- `--accesslog`
    - Enables the access log


## Enable API
```yaml
      # Admin Flags
      - "--api"
```
This enables the Admin api, and the dashboard.

## Ports to listen on
```yaml
      # HTTP
      - "--entryPoints.web.address=:80"
      # HTTPS
      - "--entrypoints.websecure.address=:443"
```
`--entryPoints.web.address` sets the listening port. The one described here is named web. If you want to change that name you can rename it to whatever you need. In the example above you can see a similar one named `websecure`. In this example, `web` is setup to accept traffic on port 80 (which is typically http),  `websecure` is setup to accept traffic on port 80 (which is typically https). 

## LetsEncrypt - HTTPS Certificate Verification Setup
```yaml
     # HTTPS - Create Certificates
      - "--certificatesresolvers.le.acme.tlschallenge=true"
      - "--certificatesresolvers.le.acme.email=${DOCKER_TRAEFIK_LETSENCRYPT_EMAIL}"
      - "--certificatesresolvers.le.acme.storage=/letsencrypt/acme.json"
      - "--certificatesresolvers.le.acme.httpchallenge=false"
      - "--certificatesresolvers.le.acme.httpchallenge.entrypoint=web"

      # HTTPS - Create Certificates - dev
      - "--certificatesresolvers.leDev.acme.tlschallenge=true"
      - "--certificatesresolvers.leDev.acme.email=${DOCKER_TRAEFIK_LETSENCRYPT_EMAIL}"
      - "--certificatesresolvers.leDev.acme.storage=/letsencrypt/acme_dev.json"
      - "--certificatesresolvers.leDev.acme.httpchallenge=false"
      - "--certificatesresolvers.leDev.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.leDev.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"
```
Similarly to the configuration on the ports. We also have the ability to name the certificate resolvers. The certificate resolvers in this configuration are named `le` and `leDev`. The Dev configuration should be used if you are testing something, and the certificate may not be persisted. Letsencrypt has a rate limit in its "Prod"  API. It has a lot less restrictive limit in place for staging. 

- `--certificatesresolvers.le.acme.tlschallenge=true` - Enables the tls challenge
- `--certificatesresolvers.le.acme.email=${DOCKER_TRAEFIK_LETSENCRYPT_EMAIL}` - Sets the email that will be used to register with letsencrypt. 
- `--certificatesresolvers.le.acme.storage=/letsencrypt/acme.json` - Sets the location and file location to save the certificates that are generated with this certificate resolver.
- `--certificatesresolvers.leDev.acme.httpchallenge=false` - Enable the http challenge
- `--certificatesresolvers.leDev.acme.httpchallenge.entrypoint=web` - the entry point to use if httpchallenge is enabled.
- `--certificatesresolvers.leDev.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory` - Sets the staging url to get a less restrictive rate limit

## LetsEncrypt - DNS Certificate Verification Setup
```yaml
      # HTTPS - Create Certificates - dns
      - "--certificatesresolvers.ledns.acme.dnschallenge=true"
      - "--certificatesresolvers.ledns.acme.email=${DOCKER_TRAEFIK_LETSENCRYPT_EMAIL}"
      - "--certificatesresolvers.ledns.acme.storage=/letsencrypt/acme_dns.json"
      - "--certificatesresolvers.ledns.acme.dnschallenge.provider=namecheap"
      - "--certificatesresolvers.ledns.acme.dnschallenge.delaybeforecheck=30"
      # HTTPS - Create Certificates - dns - dev
      - "--certificatesresolvers.lednsDev.acme.dnschallenge=true"
      - "--certificatesresolvers.lednsDev.acme.email=${DOCKER_TRAEFIK_LETSENCRYPT_EMAIL}"
      - "--certificatesresolvers.lednsDev.acme.storage=/letsencrypt/acme_dev_dns.json"
      - "--certificatesresolvers.lednsDev.acme.dnschallenge.provider=namecheap"
      - "--certificatesresolvers.lednsDev.acme.dnschallenge.delaybeforecheck=30"
      - "--certificatesresolvers.lednsDev.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"
```
- `--certificatesresolvers.ledns.acme.dnschallenge=true` - Enables the dns challenge
- `--certificatesresolvers.ledns.acme.email=${DOCKER_TRAEFIK_LETSENCRYPT_EMAIL}` - Sets the email that will be used to register with letsencrypt.
- `--certificatesresolvers.ledns.acme.storage=/letsencrypt/acme_dns.json` -  Sets the location and file location to save the certificates that are generated with this certificate resolver.
- `--certificatesresolvers.lednsDev.acme.dnschallenge.provider=namecheap` - Your dns provider
- `--certificatesresolvers.lednsDev.acme.dnschallenge.delaybeforecheck=30` - Sets the amount of time to wait until letsencrypt should check for the dns challenge to be setup.
- `--certificatesresolvers.lednsDev.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory` - Sets the staging url to get a less restrictive rate limit

## Configuration Providers
### Docker Provider
```yaml
      # Providers
      ## Docker Provider
      - "--providers.docker"
      - "--providers.docker.exposedbydefault=false"
```
- `--providers.docker` - Enables the docker provider
- `--providers.docker.exposedbydefault=false` - Sets whether any docker container that is deployed should be enabled. 

### File Provider
```yaml
      ## File Provider for Custom local services
      - "--providers.file=true"
      - "--providers.file.directory=/traefikconfig/routes"
      - "--providers.file.watch=true"
```
- `--providers.file=true` - Enable the file provider
- `--providers.file.directory=/traefikconfig/routes` - Where to watch 
- `--providers.file.watch=true` - Should we only watch the folder or only update when traefik is started.


