# Webserver configuration

Note: This document is part of the documentation "[Howto build a website on wordpress](./README.md)".
You might want to start from the very beginning of it.

## Webserver configuration overview

In the last chapter I did all the basic configuration my webserver machine required. Now I am ready to install and configure my first services. This includes:

- install a reverse proxy server
- install Wordpress from docker
- setup SSL encryption with LetsEncrypt
- add scripts & cronjobs for my local Wordpress backup

## Install and configure HaProxy as reverse proxy service

On my server I will run many services in the future. This includes my Wordpress based website, but also a Nextcloud and other websites. I will do myself a favor, if I now add a "reverse proxy" service in front of all of them. This service will:

- do everything about SSL certificates and https for all my services
- do all the routing for several domains / subdomains for all my services
- do some load balancing in the far future too (I hope my website traffic will require this!!).

Even if I know, that many users prefer Nginx, I choose [HAProxy](http://www.haproxy.org/) to be my reverse proxy service.
- docker-compose.yml
- configfile
- check success

## Install Wordpress from docker

- Wordpress and Database
- docker-compose.yml
