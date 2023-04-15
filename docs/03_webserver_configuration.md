# Webserver configuration

Note: This document is part of the documentation "[Howto build a website on wordpress](../README.md)".
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

Even if I know, that many users prefer Nginx, I choose [HAProxy](http://www.haproxy.org/) to be my reverse proxy service. Attached to this git repository You'll find:
- a docker-compose.yml to start haproxy, matomo web analytics and wordpress as docker containers
- the haproxy configfile and dockerfile 

I uploaded these files into this [GitHub repository](https://github.com/Entwicklungsleiter/howto_build_a_website_on_wordpress) and cloned it on my server. After some adjustments (in example a new database password), I started the docker containers:

```shell
cd /where/you/want/your/containers
git clone https://github.com/Entwicklungsleiter/howto_build_a_website_on_wordpress.git
cd docker
docker-compose up --build -d         # start all containers in docker-compose file
docker-compose ps                    # check containers are running and non is endless restarting
docker-compose logs <containername>  # get more information if a container is not "up"
```

## generate SSL certificate with letsencrypt

In the early days of the web we had to buy "SSL certificates" signed by "certificate authorities" to make our websites available using https. The certificates of these important companies (aka "root certificates") were integrated in and accepted by all major webbrowsers. A website was connsidered being "safe through https" of one of these authorities signed Your websites SSL certificate.
Today this process is still driving https traffic. But today there is one more independent certificate authority integrated into all major webbrowsers, letsencrypt. To create Your own https certificate and get it signed by them, You need to follow these steps:

The commandline software to work with letsencrypt is easy to install from Ubuntu package repository:
```shell
sudo apt install certbot
```

- test if a first website is available already via browser
- run certbot manually for all domains
- add certificate to haproxy configuration
- restart haproxy
