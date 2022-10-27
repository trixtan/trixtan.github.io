---
layout: post
title:  "Exposing CDT.cloud with Nginx"
date:   2022-10-27 09:58:54 +0100
---
To expose CDT.cloud and make it accessible from any browser, I opted fornthe following stack:
- NGINX as web server, configured for reverse proxying requests to CDT.cloud
- LetsEncrypt to get free SSL certificates
 
## Configuring NGINX and LetsEncrypt
To simplify this task I used [NGINX with LetEncrypt proxy companion][ngpc].
I put the final configuration in a [docker compose configuration file](https://github.com/trixtan/nginx-proxy-environment/blob/7418e84f6a02751285a1d1e825baae898b544a81/nginx-proxy-compose.yaml).
To load the nginx environment:
```sh
docker-compose -f nginx-proxy-compose.yaml up -d
```
After that, I can manage the reverse proxy configuration at the level of the web application I want to expose, using environment variables.

## Attaching CDT.cloud to nginx-proxy
I created an additional docker-compose configuration file for CDT.cloud, which looks like this:

```yaml
version: '2.2'

services:
        ads-cloud:
                restart: always
                image: cdt-cloud-blueprint:latest
                environment:
                        - VIRTUAL_HOST=.ads.n-ri.co
                        - VIRTUAL_PORT=3000
                        - LETSENCRYPT_HOST=ads.n-ri.co
                networks:
                        - proxy-tier
networks:
        proxy-tier:
                external:
                        name: nginx-proxy
```
Noteworthy aspects of this configuration:
- `environment`
	- `VIRTUAL_HOST=.ads.n-ri.co`: with this I am asking nginx-proxy to create a reverse proxy configuration to serve the domain __*.ads.n-ri.co__ with the cdt blueprint application. Note the dot at the beginning of the pattern. That is necessary to serve any url in the form __aaa.bbb.ads.n-ri.co__, as explained in the NGINX `[server_name](https://nginx.org/en/docs/http/server_names.html)` documentation.

## DNS Configuration
The last thing to do is to enable Internet to correctly route all requests to the right IP address.
I added the following two rows to my dns configuration.
```sh
ads                IN A 88.198.161.42
*.ads              IN CNAME ads
```
Note the use of a wildcard host name, necessary to route requests in the form `__aaa.bbb__.ads.`.

[ngpc]: https://github.com/jwilder/docker-letsencrypt-nginx-proxy-companion
