# Web Proxy using Docker, NGINX and Let's Encrypt

With this repo you will be able to set up your server with multiple sites using a single NGINX proxy to manage your connections, automating your apps container (port 80 and 443) to auto renew your ssl certificates with Let´s Encrypt.

Something like:

![Web Proxy environment](https://github.com/evertramos/images/raw/master/webproxy.jpg)


## Why use it?

Using this set up you will be able start a production environment in a few seconds. And to start new web project simply start new containers the option `-e VIRTUAL_HOST=your.domain.com` and will be ready to go, if you want to use SSL (Let's Encrypt) just add the tag `-e LETSENCRYPT_HOST=your.domain.com`. Done!

Easy and trustworthy!


## Prerequisites

In order to use this compose file (docker-compose.yml) you must have:

1. docker (https://docs.docker.com/engine/installation/)
2. docker-compose (https://docs.docker.com/compose/install/)


## How to use it

1. Clone this repository:

```bash
git clone https://github.com/evertramos/docker-compose-letsencrypt-nginx-proxy-companion.git
```

2. Make a copy of our `.env.sample` and rename it to `.env`:

Update this file with your preferences.

```
#
# docker-compose-letsencrypt-nginx-proxy-companion
# 
# A Web Proxy using docker with NGINX and Let's Encrypt
# Using the great community docker-gen, nginx-proxy and docker-letsencrypt-nginx-proxy-companion
#
# This is the .env file to set up your webproxy enviornment

#
# Your local containers NAME
#
NGINX_WEB=nginx-web
DOCKER_GEN=nginx-gen
LETS_ENCRYPT=nginx-letsencrypt

#
# Your external IP address
#
IP=0.0.0.0

#
# Default Network
#
NETWORK=webproxy

#
# Service Network
#
# This is optional in case you decide to add a new network to your services containers
#SERVICE_NETWORK=webservices

#
# NGINX file path
#
NGINX_FILES_PATH=/path/to/your/nginx/data
```

4. Run our start script

```bash
# ./start.sh
```

Your proxy is ready to go!

## Starting your web containers

After following the steps above you can start new web containers with port 80 open and add the option `-e VIRTUAL_HOST=your.domain.com` so proxy will automatically generate the reverse script in NGINX Proxy to forward new connections to your web/app container, as of:

```bash
docker run -d -e VIRTUAL_HOST=your.domain.com \
              --network=webproxy \
              --name my_app \
              httpd:alpine
```

To have SSL in your web/app you just add the option `-e LETSENCRYPT_HOST=your.domain.com`, as follow:

```bash
docker run -d -e VIRTUAL_HOST=your.domain.com \
              -e LETSENCRYPT_HOST=your.domain.com \
              -e LETSENCRYPT_EMAIL=your.email@your.domain.com \
              --network=webproxy \
              --name my_app \
              httpd:alpine
```

> You don´t need to open port *443* in your container, the certificate validation is managed by the web proxy.


> Please note that when running a new container to generate certificates with LetsEncrypt (`-e LETSENCRYPT_HOST=your.domain.com`), it may take a few minutes, depending on multiples circunstances.

## Futher Options

1. Basic Authentication Support

In order to be able to secure your virtual host with basic authentication, you must create a file named as its equivalent VIRTUAL_HOST variable on directory ${NGINX_FILES_PATH}/htpasswd/$VIRTUAL_HOST.

After set up this webproxy, you will do the following:
```bash
sudo sh -c "echo -n 'user_name:' >> ${NGINX_FILES_PATH}/your_domain.com"
sudo sh -c "openssl passwd -apr1 >> ${NGINX_FILES_PATH}/your_domain.com"
```
> Please substitute the `${NGINX_FILES_PATH}` with your path information and your virtual host name as `your_domain.com`.

2. Using multiple networks

If you want to use more than one network to better organize your environment you could set the option `SERVICE_NETWORK` in our `.env.sample` or you can just create your own network and attach all your containers as of:

```bash
docker network create myownnetwork
docker network connect myownnetwork nginx-web
docker network connect myownnetwork nginx-gen
docker network connect myownnetwork nginx-letsencrypt
```

## Testing your proxy with scripts preconfigured 

1. Run the script `test.sh` informing your domain already configured in your DNS to point out to your server as follow:

```bash
# ./test_start.sh your.domain.com
```

or simply run:

```bash
 docker run -dit -e VIRTUAL_HOST=your.domain.com --network=webproxy --name test-web httpd:alpine
```

Access your browser with your domain!

To stop and remove your test container run our `stop_test.sh` script:

```bash
# ./test_stop.sh
```

Or simply run:

```bash
docker stop test-web && docker rm test-web 
```

## Production Environment using Web Proxy and Wordpress

1. [wordpress-docker-letsencrypt](https://github.com/evertramos/wordpress-docker-letsencrypt)
2. [docker-portainer-letsencrypt](https://github.com/evertramos/docker-portainer-letsencrypt)

In this repo you will find a docker-compose file to start a production environment for a new wordpress site.

## Credits

Without the repositories below this webproxy wouldn´t be possible.

Credits goes to:
- nginx-proxy [@jwilder](https://github.com/jwilder/nginx-proxy)
- docker-gen [@jwilder](https://github.com/jwilder/docker-gen)
- docker-letsencrypt-nginx-proxy-companion [@JrCs](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion)


### Special thanks to:

- [@buchdag](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion/pull/226#event-1145800062)
- [@fracz](https://github.com/fracz) - Update repo to use `.env` file
