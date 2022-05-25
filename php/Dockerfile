FROM php:7.4.29-fpm-bullseye

MAINTAINER wildcore.tools <dev@wildcore.tools>

# Install php and packages
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    git \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
	libpng-dev \
	libonig-dev \
	libzip-dev \
    libssh2-1 \
    libssh2-1-dev \
    libmcrypt-dev \
	libyaml-dev \
	libcurl3-dev \
    libssl-dev \
	libsnmp-dev \
	libxml2-dev \
	libedit-dev \
    libmemcached-dev  \
    zlib1g-dev \
	unzip && \
    apt-get clean autoclean && \
    apt-get autoremove --yes && \
    rm -rf /var/lib/{apt,dpkg,cache,log}/


RUN docker-php-ext-install -j$(nproc) iconv mbstring mysqli pdo_mysql zip json curl snmp xml opcache sockets
RUN pecl install memcached yaml mcrypt xdebug redis ssh2-1.3.1 \
    && docker-php-ext-enable memcached mcrypt yaml xdebug opcache sockets redis zip ssh2

#Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

#Copy configuration and scripts
ADD php.ini /usr/local/etc/php/conf.d/40-custom.ini
ADD www.conf /usr/local/etc/php-fpm.d/www.conf

#Prepare env
RUN rm /usr/local/etc/php-fpm.d/zz-docker.conf
ADD docker-entrypoint.sh /docker-entrypoint.sh
RUN chmod +x /docker-entrypoint.sh
RUN mkdir /log && mkdir /www
ENV TERM=xterm-256color

WORKDIR /www


ENTRYPOINT /docker-entrypoint.sh
