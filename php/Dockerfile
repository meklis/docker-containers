FROM php:7.4.33-fpm-bullseye

MAINTAINER wildcore.tools <dev@wildcore.tools>

# Debian bullseye is EOL - use archive repositories
RUN sed -i -e 's#deb.debian.org#archive.debian.org#g' -e '/bullseye-updates/d' /etc/apt/sources.list

# Install php and packages
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    git \
    libc-bin \
    libc6 \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
	libpng-dev \
	libonig-dev \
	libzip-dev \
    libmcrypt-dev \
	libyaml-dev \
	libcurl4-openssl-dev \
	libwebp-dev \
    libssl-dev \
	libsnmp-dev \
	libxml2-dev \
	libedit-dev \
    libmemcached-dev  \
    zlib1g-dev \
    libldap2-dev  \
    snmp \
    socat \
    ffmpeg \
    libmagickwand-dev ghostscript \
    unzip && \
    apt-get clean autoclean && \
    apt-get autoremove --yes && \
    rm -rf /var/lib/{apt,dpkg,cache,log}/

# Rebuild PHP with the PHP security team backport of CVE-2026-109618 and CVE-2026-109619 fixes for 7.4
# https://www.php.net/archive/2026.php#2026-10-21-5
# https://github.com/php/php-src/tree/PHP-7.4-CVE-2026-109618-109619
COPY patches/CVE-2026-109618-109619.patch /usr/src/
RUN apt-get update && apt-get install -y $PHPIZE_DEPS patch libargon2-dev libreadline-dev libsodium-dev libsqlite3-dev \
    && docker-php-source extract \
    && cd /usr/src/php \
    && patch -p1 < /usr/src/CVE-2026-109618-109619.patch \
    && export CFLAGS="$PHP_CFLAGS" CPPFLAGS="$PHP_CPPFLAGS" LDFLAGS="$PHP_LDFLAGS" \
    && ./configure $(php-config --configure-options) \
    && make -j$(nproc) \
    && make install \
    && make clean \
    && cd / && docker-php-source delete \
    && rm /usr/src/CVE-2026-109618-109619.patch \
    && apt-get clean autoclean && \
    rm -rf /var/lib/{apt,dpkg,cache,log}/

# libssh2 1.11.0 built with OpenSSL - bullseye ships 1.9.0 (libgcrypt) without curve25519, ed25519, ecdsa, aes-gcm etc.,
# which newer network devices require
RUN cd /tmp && wget https://www.libssh2.org/download/libssh2-1.11.0.tar.gz \
    && echo "3736161e41e2693324deb38c26cfdc3efe6209d634ba4258db1cecff6a5ad461 *libssh2-1.11.0.tar.gz" | sha256sum -c - \
    && tar -xzf libssh2-1.11.0.tar.gz && cd libssh2-1.11.0 \
    && ./configure --prefix=/usr/local --with-crypto=openssl && make -j"$(nproc)" && make install && ldconfig \
    && cd / && rm -rf /tmp/libssh2-1.11.0*

RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp \
     && docker-php-ext-install -j$(nproc) iconv mbstring mysqli pdo_mysql zip json curl snmp xml opcache sockets ldap posix pcntl exif gd
# Versions are pinned to the ones in meklis/php-wca:7.4-240726-libssh2.
# yaml and imagick are installed from direct URLs - pecl fails to resolve them by name ("does not have REST xml available")
RUN pecl install memcached-3.4.0 https://pecl.php.net/get/yaml-2.3.0.tgz mcrypt-1.0.9 xdebug-3.1.5 redis-6.3.0 ssh2-1.3.1 https://pecl.php.net/get/imagick-3.8.1.tgz \
    && docker-php-ext-enable memcached mcrypt yaml xdebug opcache sockets redis zip ssh2 posix pcntl exif imagick gd

# swoole 4.8.12 is no longer available on PECL - build from GitHub sources
RUN curl -fsSL https://github.com/swoole/swoole-src/archive/refs/tags/v4.8.12.tar.gz | tar xz -C /tmp \
        && cd /tmp/swoole-src-4.8.12 && phpize && ./configure && make -j$(nproc) && make install \
        && cd / && rm -rf /tmp/swoole-src-4.8.12 \
        && docker-php-ext-enable swoole
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
