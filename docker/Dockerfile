FROM php:7.2-cli

LABEL MAINTAINER="Pavel Livinskiy <plivinskiy@tagwork-one.de>"

RUN apt-get update \
  && apt-get install -y \
    libfreetype6-dev \
    libicu-dev \
    libjpeg62-turbo-dev \
    libsodium-dev \
    libzip-dev \
    libpng-dev \
    libxslt1-dev \
    openssl \
    sudo \
    cron \
    rsyslog \
    default-mysql-client \
    git \
    unzip \
    procps

# Configure the gd library
RUN docker-php-ext-configure \
  gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/

# Install required PHP extensions

RUN docker-php-ext-install \
  sockets \
  dom \
  gd \
  intl \
  mbstring \
  pdo_mysql \
  mysqli \
  xsl \
  zip \
  soap \
  bcmath \
  iconv \
  simplexml \
  exif \
  opcache \
  sodium

VOLUME /root/.composer/cache
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer  --version=1.10.16

RUN echo 'memory_limit=-1' >> /usr/local/etc/php/conf.d/zz-magento.ini

COPY files/store-config.php /var/store-config.php

CMD ["bash"]
