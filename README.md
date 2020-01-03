# docker-php-apache-oracle

Docker image with apache, php 7.4, oci8, pdo_oci, and oracle instant_client 12.

## Usage
``` 
docker run -e TNS_ADMIN=/root -v /path/to/local/tnsnames.ora:/root/tnsnames.ora -v /path/to/webfiles:/var/www/html fallenpixel/docker-php-apache-oracle 
```
Reivew [PHP on Dockerhub](https://hub.docker.com/_/php) for additional options regarding the webserver.

Forked from https://github.com/vitoo/docker-php-oci8
