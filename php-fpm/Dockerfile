#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#
ARG PHP_VERSION=8.1
FROM php:${PHP_VERSION}-fpm

LABEL maintainer="zhangatle"

ENV DEBIAN_FRONTEND noninteractive

ARG CHANGE_SOURCE=true
RUN if [ ${CHANGE_SOURCE} = true ]; then \
    # 更改为国内阿里源提高软件安装速度
    sed -i 's/deb.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
    ;fi

# 安装额外扩展的工具，执行完拥有以下命令，docker-php-ext-configure, docker-php-ext-install, and docker-php-ext-enable
RUN docker-php-source extract && docker-php-source delete

# always run apt update when start and after add new source list, then clean up at end
RUN set -xe; \
    apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    apt-get install -yqq apt-utils gnupg2 git libzip-dev unzip zip && \
    docker-php-ext-configure zip && docker-php-ext-install zip && \
    php -m | grep -q 'zlib'

###########################################################################
# PHP REDIS EXTENSION
###########################################################################
ARG INSTALL_PHPREDIS=false
RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
    # Install Php Redis Extension
    pecl install -o -f redis \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis && \
    php -m | grep -q 'redis' \
    ;fi

###########################################################################
# SWOOLE
###########################################################################
ARG INSTALL_SWOOLE=false
RUN set -eux; \
    if [ ${INSTALL_SWOOLE} = true ]; then \
    # Install Php Swoole Extension
    pecl install swoole; \
    docker-php-ext-enable swoole; \
    php -m | grep -q 'swoole'; \
    fi

###########################################################################
# AMQP
###########################################################################
ARG INSTALL_AMQP=false
RUN set -eux; \
    if [ ${INSTALL_AMQP} = true ]; then \
    # # Install the amqp extension
    apt-get -yqq install librabbitmq-dev; \
    pecl install amqp; \
    docker-php-ext-enable amqp; \
    php -m | grep -oiE '^amqp$'; \
    fi

###########################################################################
# MongoDB:
###########################################################################
ARG INSTALL_MONGO=false
RUN if [ ${INSTALL_MONGO} = true ]; then \
    # Install the mongodb extension
    pecl install mongodb; \
    docker-php-ext-enable mongodb; \
    php -m | grep -oiE '^mongodb$'; \
    fi

###########################################################################
# Human Language and Character Encoding Support:
###########################################################################
ARG INSTALL_INTL=false
RUN if [ ${INSTALL_INTL} = true ]; then \
    # Install intl and requirements
    apt-get install -yqq zlib1g-dev libicu-dev g++ && \
    docker-php-ext-configure intl && \
    docker-php-ext-install intl \
    ;fi

###########################################################################
# Composer
###########################################################################
RUN set -eux; \
    cd /tmp && \
    php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');" && \
    php composer-setup.php && \
    php -r "unlink('composer-setup.php');" && \
    mv composer.phar /usr/local/bin/composer

###########################################################################
# Check PHP version:
###########################################################################
RUN set -xe; php -v | head -n 1 | grep -q "PHP ${PHP_VERSION}."

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#
COPY ./laravel.ini /usr/local/etc/php/conf.d
COPY ./xlaravel.pool.conf /usr/local/etc/php-fpm.d/

USER root

# Clean up
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Configure non-root user.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN groupmod -o -g ${PGID} www-data && usermod -o -u ${PUID} -g www-data www-data

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000