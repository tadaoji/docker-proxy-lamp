# [ENG] README.md: docker-proxy-lamp

# README.md: docker-proxy-lamp

# ToC
# Overview
Use nginx's proxy (steveltn/https-portal) to set up multiple web servers on one host. Here, we use Let's encrypt to set up an SSL-enabled LAMP configuration. By default, php-apache, mysql, wordpress, and phpmyadmin are automated and built.

# Required environment
Linux (tested on CentOS7)
docker-compose
docker

# Configuration method
Edit docker-compose.yml
A single coherent section is separated by ############## (comment)

## https-portal
[SteveLTN/https-portal : https://github.com/SteveLTN/https-portal](https://github.com/SteveLTN/https-portal)
using.

## restart: always
It is recommended to comment it out when testing the behavior.
In the case of always, SSL issue will be attempted by Let's encrypt every time when restart is continued for some reason, and it may be trapped by the SSL issue request limit of Let's encrypt.

### environment
#### DOMAINS
Configure the proxy settings.
The basic format is as follows.
In this example, we will connect to two web servers (php-apache_1 and wordpress-app_1).
```
'test.example.fqdn -> http://php-apache_1, wp-test.example.fqdn -> http://wordpress-app_1'
```
The web server name specification matches the service name.
The web server name matches the service name because it is specified in the same network and DNS resolution is done by the service name.

#### STAGE
In the case of production, Let's encrypt will issue an SSL that can be used.
In the case of local, it will be a self-certificate.
For testing, local is recommended to avoid reaching the Let's encrypt limit.
For more information, see https://github.com/SteveLTN/https-portal.

### depends_on
As SSL is issued by Let's encrypt, launch the proxy while the web server is up. It is recommended to add it here if you are going to launch any additional services.

## LAMP(php-apache, mysql-db)
You can create any number of LAMP servers by appending more sections to this section.
If you want to add more, copy and rewrite the following sections to avoid duplication
php-apache
* service name
* volumes (host side)
* depends_on

mysql-db
* service name
* volumes (host side)
* add volumes to be used by mysql-db in volumes section

### php-apache
Using the official docker image for php
[php : https://hub.docker.com/_/php](https://hub.docker.com/_/php)

#### build
##### args
By specifying DOCKER_LOCATION, you can set the timezone for PHP running in the container. This will match the directory structure under /usr/share/zoneinfo/ in the container being created.
If it is not specified, it will be set to UTC.

#### volumes
Set the public directory to . /lamp_1/public_html.
Set the log directory to . /lamp_1/apache_logs.
These can be rewritten as needed.

#### networks
This needs to be a common setting for all containers.

### mysql-db
mysql will use version 5:Latest.

#### build
##### args
DOCKER_LOCATION can be used to set the timezone for PHP running in the container. This will match the directory structure under /usr/share/zoneinfo/ in the container being created.
If it is not specified, it will be set to UTC.

If MYSQL_ROOT_PASSWORD is set, /root/.my.cnf will be created in the container. This eliminates the need for a password when using the mysql command in the container.

#### environment
You can set any environment you like.


## phpmyadmin
### https-portal-phpmyadmin
#### ports
Due to the nature of phpmyadmin, it is not necessary to make it available to the public, so a random port is recommended.
In order to use only arbitrary ports, create a separate container for https-portal from the one for lamp.

#### environment
##### STAGE
Since both 80 and 443 are not used, the Let's encrypt authentication will not pass.
Therefore, local is used as a self-certificate.

### phpmyadmin
[https://hub.docker.com/r/phpmyadmin/phpmyadmin/](https://hub.docker.com/r/phpmyadmin/phpmyadmin/)
Use the above image.

#### environment
You can decide if you want to fix the DB that phpmyadmin connects to.
By setting PMA_ARBITARY to 1, you can choose any server when you login.
By setting PMA_HOST to any DB, it will be fixed.
For details, see [https://hub.docker.com/r/phpmyadmin/phpmyadmin/](https://hub.docker.com/r/phpmyadmin/phpmyadmin/)

#### How to login
To specify the DB connection destination on the login screen, enter the service name of the DB.
The service name will be DNS resolved by the Docker network function.

# Dockerfile
## File structure
```
├─ docker-lamp
│ ├─ mysql
│ │└─ Dockerfile
│└─ php-apache
│└─ Dockerfile
```

## mysql
### general_log
If you want to output mysql's general_log to a file from within the container, uncomment the following.
```
general_log=1\n\ # Note that the log file will be large.
general_log_file=/var/log/mysql/query.log\n\ # Note that this will increase the size of the log file.
```
This setting will be written to /etc/mysql/my.cnf in the container.
However, note that the file will be large.

If you use the volume option of docker to allow the host to handle the log file, you may need to set 77x in the directory permission. If you don't do this, mysql will fail to write the log file and the container will be down.

# Credits
[https-portal by SteveLTN](https://github.com/SteveLTN)
[phpmyadmin by Docker Official Images](https://hub.docker.com/_/phpmyadmin)
[php by Docker Official Images](https://hub.docker.com/_/php)
[mysql by Docker Official Images](https://hub.docker.com/_/mysql)