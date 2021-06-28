FROM php:7.4-fpm
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
	libmcrypt-dev \
	libyaml-dev \
	libcurl3-dev \
	libsnmp-dev \
	libxml2-dev \
	libedit-dev \
    && pecl install mcrypt-1.0.3 \
	&& docker-php-ext-enable mcrypt \
    && docker-php-ext-install -j$(nproc) iconv mbstring mysqli pdo_mysql zip json curl snmp xml readline

# Install php binary libraries
RUN  pecl install yaml && echo "extension=yaml.so" > /usr/local/etc/php/conf.d/ext-yaml.ini && docker-php-ext-enable yaml
RUN apt-get install -y libmemcached-dev zlib1g-dev \
    && pecl install memcached \
    && docker-php-ext-enable memcached

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
