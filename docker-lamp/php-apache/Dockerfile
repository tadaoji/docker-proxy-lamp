FROM php:7-apache

ENV APACHE_LOG_DIR=/opt/log
ARG DOCKER_LOCATION

RUN ln -sf /usr/share/zoneinfo/${DOCKER_LOCATION:-UTC} /etc/localtime
RUN apt-get update \
&& apt-get install -y \
git \
zip \
unzip \
vim \
libpng-dev \
libpq-dev \
&& docker-php-ext-install pdo_mysql \
&& cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \
&& mkdir -p /opt/log


