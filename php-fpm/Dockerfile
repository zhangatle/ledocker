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
# BZ2:
###########################################################################
ARG INSTALL_BZ2=false
RUN if [ ${INSTALL_BZ2} = true ]; then \
    apt-get -yqq install libbz2-dev; \
    docker-php-ext-install bz2 \
    ;fi

###########################################################################
# GnuPG:
###########################################################################
ARG INSTALL_GNUPG=false
RUN if [ ${INSTALL_GNUPG} = true ]; then \
    apt-get -yq install libgpgme-dev; \
    pecl install gnupg; \
    docker-php-ext-enable gnupg; \
    php -m | grep -q 'gnupg'; \
    fi

###########################################################################
# GMP (GNU Multiple Precision):
###########################################################################
ARG INSTALL_GMP=false
RUN if [ ${INSTALL_GMP} = true ]; then \
    # Install the GMP extension
    apt-get install -yqq libgmp-dev && \
    docker-php-ext-install gmp \
    ;fi

###########################################################################
# SOAP:
###########################################################################
ARG INSTALL_SOAP=false
RUN if [ ${INSTALL_SOAP} = true ]; then \
    # Install the soap extension
    rm /etc/apt/preferences.d/no-debian-php && \
    apt-get -y install libxml2-dev php-soap && \
    docker-php-ext-install soap \
    ;fi

###########################################################################
# XSL:
###########################################################################
ARG INSTALL_XSL=false
RUN if [ ${INSTALL_XSL} = true ]; then \
    # Install the xsl extension
    apt-get -y install libxslt-dev && \
    docker-php-ext-install xsl \
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
# bcmath:
###########################################################################
ARG INSTALL_BCMATH=false
RUN if [ ${INSTALL_BCMATH} = true ]; then \
    # Install the bcmath extension
    docker-php-ext-install bcmath \
    ;fi

###########################################################################
# Opcache:
###########################################################################
ARG INSTALL_OPCACHE=false
RUN if [ ${INSTALL_OPCACHE} = true ]; then \
    docker-php-ext-install opcache \
    ;fi
# Copy opcache configration
COPY ./opcache.ini /usr/local/etc/php/conf.d/opcache.ini

###########################################################################
# Exif:
###########################################################################
ARG INSTALL_EXIF=false
RUN if [ ${INSTALL_EXIF} = true ]; then \
    # Enable Exif PHP extentions requirements
    docker-php-ext-install exif \
    ;fi

###########################################################################
# Mysqli Modifications:
###########################################################################
ARG INSTALL_MYSQLI=false
RUN if [ ${INSTALL_MYSQLI} = true ]; then \
    docker-php-ext-install mysqli \
    ;fi

###########################################################################
# pgsql
###########################################################################
ARG INSTALL_PGSQL=false
RUN if [ ${INSTALL_PGSQL} = true ]; then \
    # Install the pgsql extension
    docker-php-ext-install pgsql \
    ;fi

###########################################################################
# ImageMagick:
###########################################################################
USER root
ARG INSTALL_IMAGEMAGICK=false
ARG IMAGEMAGICK_VERSION=latest
ENV IMAGEMAGICK_VERSION ${IMAGEMAGICK_VERSION}

RUN if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
    apt-get install -yqq libmagickwand-dev imagemagick && \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
    cd /tmp && \
    if [ ${IMAGEMAGICK_VERSION} = "latest" ]; then \
    git clone https://github.com/Imagick/imagick; \
    else \
    git clone --branch ${IMAGEMAGICK_VERSION} https://github.com/Imagick/imagick; \
    fi && \
    cd imagick && \
    phpize && \
    ./configure && \
    make && \
    make install && \
    rm -r /tmp/imagick; \
    else \
    pecl install imagick; \
    fi && \
    docker-php-ext-enable imagick; \
    php -m | grep -q 'imagick' \
    ;fi

###########################################################################
# YAML:
###########################################################################
USER root
ARG INSTALL_YAML=false

RUN if [ ${INSTALL_YAML} = true ]; then \
    apt-get install -yqq libyaml-dev; \
    echo '' | pecl install yaml \
    && docker-php-ext-enable yaml \
    ;fi

###########################################################################
# MySQL Client:
###########################################################################
USER root
ARG INSTALL_MYSQL_CLIENT=false
RUN if [ ${INSTALL_MYSQL_CLIENT} = true ]; then \
    apt-get -y install default-mysql-client \
    ;fi

###########################################################################
# FFMPEG:
###########################################################################
USER root
ARG INSTALL_FFMPEG=false
RUN if [ ${INSTALL_FFMPEG} = true ]; then \
    apt-get -y install ffmpeg \
    ;fi

#####################################
# wkhtmltopdf:
#####################################
USER root
ARG INSTALL_WKHTMLTOPDF=false
ARG WKHTMLTOPDF_VERSION=0.12.6-1

RUN if [ ${INSTALL_WKHTMLTOPDF} = true ]; then \
    ARCH=$(arch | sed s/aarch64/arm64/ | sed s/x86_64/amd64/) \
    && apt-get install -yqq \
    libxrender1 \
    libfontconfig1 \
    libx11-dev \
    libjpeg62 \
    libxtst6 \
    fontconfig \
    libjpeg62-turbo \
    xfonts-base \
    xfonts-75dpi \
    wget \
    && wget "https://github.com/wkhtmltopdf/packaging/releases/download/${WKHTMLTOPDF_VERSION}/wkhtmltox_0.12.6-1.stretch_${ARCH}.deb" \
    && dpkg -i "wkhtmltox_${WKHTMLTOPDF_VERSION}.stretch_${ARCH}.deb" \
    && apt -f install \
    ;fi

###########################################################################
# PHP DECIMAL:
###########################################################################
USER root
ARG INSTALL_PHPDECIMAL=false
RUN if [ ${INSTALL_PHPDECIMAL} = true ]; then \
    apt-get install -yqq libmpdec-dev \
    && pecl install decimal \
    && docker-php-ext-enable decimal \
    && php -m | grep -q 'decimal' \
    ;fi

###########################################################################
# zookeeper
###########################################################################
ARG INSTALL_ZOOKEEPER=false
RUN set -eux; \
    if [ ${INSTALL_ZOOKEEPER} = true ]; then \
    apt install -yqq libzookeeper-mt-dev; \
    curl -L -o /tmp/php-zookeeper.tar.gz https://github.com/php-zookeeper/php-zookeeper/archive/master.tar.gz; \
    mkdir -p /tmp/php-zookeeper; \
    tar -C /tmp/php-zookeeper -zxvf /tmp/php-zookeeper.tar.gz --strip 1; \
    cd /tmp/php-zookeeper; \
    phpize && ./configure && make && make install;\
    docker-php-ext-enable zookeeper; \
    php -m | grep -q 'zookeeper'; \
    fi

############################################################################
## Event:
############################################################################
USER root
ARG INSTALL_EVENT=false
RUN set -eux; \
    if [ ${INSTALL_EVENT} = true ]; then \
    curl -L -o  /tmp/libevent.tar.gz https://github.com/libevent/libevent/releases/download/release-2.1.12-stable/libevent-2.1.12-stable.tar.gz   &&\
    mkdir -p /tmp/libevent-php &&\
    tar -C /tmp/libevent-php -zxvf /tmp/libevent.tar.gz --strip 1 &&\
    cd /tmp/libevent-php &&\
    ./configure --prefix=/usr/local/libevent-2.1.12  &&\
    make &&\
    make install &&\
    rm /tmp/libevent.tar.gz &&\
    docker-php-ext-install sockets  &&\
    curl -L -o /tmp/event.tar.gz http://pecl.php.net/get/event-3.0.6.tgz &&\
    mkdir -p /tmp/event-php &&\
    tar -C /tmp/event-php -zxvf /tmp/event.tar.gz --strip 1 &&\
    cd /tmp/event-php &&\
    phpize &&\
    ./configure  --with-event-libevent-dir=/usr/local/libevent-2.1.12/ &&\
    make &&\
    make install &&\
    rm /tmp/event.tar.gz &&\
    docker-php-ext-enable event &&\
    php -m  | grep -q 'event' \
    ;fi

###########################################################################
# Downgrade Openssl:
###########################################################################
ARG DOWNGRADE_OPENSSL_TLS_AND_SECLEVEL=false
RUN if [ ${DOWNGRADE_OPENSSL_TLS_AND_SECLEVEL} = true ]; then \
    sed -i 's,^\(MinProtocol[ ]*=\).*,\1'TLSv1.2',g' /etc/ssl/openssl.cnf \
    && \
    sed -i 's,^\(CipherString[ ]*=\).*,\1'DEFAULT@SECLEVEL=1',g' /etc/ssl/openssl.cnf\
    ;fi

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