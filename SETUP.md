# Setting up Oracle, InstantClient, and PHP+Apache using Docker

## Prepare
+ Ensure that you have docker installed and working.
+ Create an account at dockerhub.com if you don't have one already. 
+ Agree to Oracle's terms of service for [Oracle Database Enterprise Edition](https://hub.docker.com/_/oracle-database-enterprise-edition) and [InstantClient](https://hub.docker.com/_/oracle-instant-client) at the DockerHub website. 
+ Create a file tnsnames.ora file. The name is unimportant, but we'll map it to our containers. I put mine in ~/.tnsnames.ora the file should contain the following: 

```
### Docker Database Connection
DOCKER=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=database)(PORT=1521))
(CONNECT_DATA=(SERVER=DEDICATED)(SID=ORCLCDB)))
### CISE Oracle Database, requires UF VPN to access. 
### Details on VPN can be found here: http://www.uflib.ufl.edu/login/vpn.html
GATOR=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=oracle.cise.ufl.edu)(PORT=1521))
(CONNECT_DATA=(SERVER=DEDICATED)(SID=orcl)))
```
+ Create a docker network for our various containers. `docker network create oraclenet`
## Oracle Database
+ Download the Oracle Database image to your computer using `docker pull store/oracle/database-instantclient:12.2.0.1-slim`
+ Create a location in your filesystem to contain the files associated with the database.

        Not storing the files in the container will allow for the container to be deleted without loosing data. I used `/opt/oracle`. This location needs to be writable for UID 54321. On linux this can be accomplished by using `chown 54321:54321 /opt/oracle`  from the root account or using sudo. 
    
+ Start the database container: `docker run -d -it --net oraclenet --restart unlesss-stopped --net oracle -e TNS_ADMIN=/root -v </path/to/\>tnsnames.ora:/root/tnsnames.ora -v /opt/oracle:/ORCL --name database store/oracle/database-instantclient:12.2.0.1-slim`
+ Use`docker ps` to ensure your database container is running.


## Oracle InstantClient
+ Use `docker pull docker pull store/oracle/database-instantclient:12.2.0.1` to download the InstantClient image.
+ Create a script to create a connection to your database. Mine looks like: 
```
#!/bin/bash
docker run --rm -e "TNS_ADMIN=/root" --net oraclenet --name InstantClient -v /home/katyl/.tnsnames.ora:/root/tnsnames.ora -it store/oracle/database-instantclient:12.2.0.1 sqlplus sys/Oradoc_db1@DOCKER as sysdba
```
+ This container will be started when you run it and removed when you exit. sys is the username and Oradoc_db1 is the default password for the sys user. If you change the password, update the script. If you create a non-sys user, you can remove the "as sysdba".
+ You can edit this script to connect to the CISE department's oracle server, replace `sys..... as sysdba` section with <CISE Username\>/<CISE Password\>@GATOR

## PHP+Apache
I looked for an existing docker container that met my needs, and didn't find one. I published an image myself, in hopes of getting what I needed. 
+ Pull the PHP container using `docker pull fallenpixel/docker-php-apache-oracle`
+ Start the container using `docker run -d -it --name webserver --net oraclenet --restart unless-stopped -e TNS_ADMIN=/root -e -e APACHE_RUN_USER=#<YOUR UID> -v  /path/to/tnsnames.ora:/root/tnsnames.ora -v /path/to/development/dir:/var/www/html fallenpixel/docker-php-apache-oracle`


## Testing

+ Create a file index.php in your development directory with the following contents:

```
<html>
<head><title>Oracle Database Test</title></head>
<body>
<?php
$connection=oci_connect("USERNAME","PASSWORD","ORCLCDB");
if ($connection                 )
    echo 'Connected to the Oracle Database.';
elseif (!$connection)
    echo 'Connection failed';
 
oci_close($connection);
?>
 </body>
</html>
```

    Be sure to replace the username and password as appropriate.
+ Use `docker inspect webserver` to find the IP address  of your webserver.
+ type the IP address in the address bar of your web browser to access the webpage we just made. If everything works, you should get the message "Connected to the Oracle Database.".

Credit:

[vitoo](https://github.com/vitoo/docker-php-oci8) Dockerfile based off his work. 

