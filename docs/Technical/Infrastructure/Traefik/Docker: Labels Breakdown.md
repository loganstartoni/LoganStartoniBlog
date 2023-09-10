---
title: Traefik - Labels Breakdown
summary: The Traefik Label Configuration.
authors:
    - Logan Startoni
date: 2023-09-10
---
# Overview
This page will break down the parts of the Label in the [docker file](Docker%3A%20All%20Together%20Now.md). This will allow you to extend it if you have to. Additional this will show you some of the configuration available to you for exposing a docker image.

Configuring the [command](Docker%3A%20Command%20Breakdown.md) to do what I wanted and the [labels](Docker%3A%20Labels%20Breakdown.md) is something that when I first dived into setting up my setup I struggled to understand. 


# Labels Breakdown
## Global HTTP Redirect
```yaml
    labels:
      # Global redirect to HTTPS
      - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)"
      - "traefik.http.routers.http-catchall.entrypoints=web"
      - "traefik.http.routers.http-catchall.middlewares=redirect-to-https"
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
```
There are two different things that you can decide the name for here: `http-catchall` and `redirect-to-https`

- `traefik.http.routers.http-catchall.rule=hostregexp('{host:.+}')` - sets what urls to match on. This particular rule uses a regular expression. 
- `traefik.http.routers.http-catchall.entrypoints=web` - Sets the entrypoint to attach to this router on. 
- `traefik.http.routers.http-catchall.middlewares=redirect-to-https` - What middlewares to use when this router is activated. This can be a list if multiple are needed. 

## Enable the Traefik Dashboard
```yaml
      # Dashboard over HTTPS
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.rule=Host(`dash.${TRAEFIK_DOMAIN}`)"
      - "traefik.http.routers.traefik.service=api@internal"
      - "traefik.http.routers.traefik.tls.certresolver=le${DOCKER_CERT_RESOLVER_SUFFIX}"
      - "traefik.http.routers.traefik.entrypoints=websecure"
      - "traefik.http.middlewares.ipwhitelist.ipwhitelist.sourcerange=${DOCKER_TRAEFIK_WHITELIST}"
      - "traefik.tcp.middlewares.ipwhitelist.ipwhitelist.sourcerange=${DOCKER_TRAEFIK_WHITELIST}"
      - "traefik.http.routers.traefik.middlewares=ipwhitelist"
```
This is how you expose a container that is exposed via docker. You can customize the name `traefik` when updating other routers. This specific rule exposes the dashboard for traefik. 

- `traefik.enable=true` - Enables the exposure of this container
- `traefik.http.routers.traefik.rule=Host('dash.${TRAEFIK_DOMAIN}')` - The exact domain to match.
- `traefik.http.routers.traefik.service=api@internal` - The service that is already configured in traefik that should be used with this configuration
- `traefik.http.routers.traefik.tls.certresolver=le${DOCKER_CERT_RESOLVER_SUFFIX}` - The certificate resolver to use to generate a certificate.
- `traefik.http.routers.traefik.entrypoints=websecure` - The entrypoint to use to access the service. 
- `traefik.http.middlewares.ipwhitelist.ipwhitelist.sourcerange=${DOCKER_TRAEFIK_WHITELIST}` - Configures the whitelist for http services
- `traefik.tcp.middlewares.ipwhitelist.ipwhitelist.sourcerange=${DOCKER_TRAEFIK_WHITELIST}` - Configure the whitelist for tcp services
- `traefik.http.routers.traefik.middlewares=ipwhitelist` - Assign the whitelist as a middleware to the traefik router. 
