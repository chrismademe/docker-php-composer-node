FROM php:7.4-apache
MAINTAINER Chris Galbraith <chris@resknow.co.uk>

ENV APACHE_DOCROOT /var/www/html/web
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data

#
# Install basic requirements
#
RUN apt-get update \
 && apt-get install -y \
 curl \
 apt-transport-https \
 git \
 build-essential \
 libssl-dev \
 wget \
 unzip \
 bzip2 \
 libbz2-dev \
 zlib1g-dev \
 libfontconfig \
 libfreetype6-dev \
 libjpeg62-turbo-dev \
 libpng-dev \
 libicu-dev \
 libxml2-dev \
 libldap2-dev \
 libmcrypt-dev \
 python-pip \
 fabric \
 jq \
 gnupg \
 && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


#
# Install Node (with NPM), and Yarn (via package manager for Debian)
#
# https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash -
RUN apt-get update && apt-get install -y nodejs
RUN npm install -g yarn

#
# Install Composer and Drush
#
ENV PATH "/composer/vendor/bin:$PATH"
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /composer
ENV COMPOSER_VERSION 1.10.8

RUN EXPECTED_CHECKSUM="$(wget -q -O - https://composer.github.io/installer.sig)" \
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
ACTUAL_CHECKSUM="$(php -r "echo hash_file('sha384', 'composer-setup.php');")" \
if [ "$EXPECTED_CHECKSUM" != "$ACTUAL_CHECKSUM" ] \
then \
    >&2 echo 'ERROR: Invalid installer checksum' \
    rm composer-setup.php \
    exit 1 \
fi \
php composer-setup.php --quiet \
RESULT=$? \
rm composer-setup.php

#
# Install additional php extensions
#
RUN docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu \
    && docker-php-ext-install -j$(nproc) \
      bcmath \
      bz2 \
      calendar \
      exif \
      ftp \
      gd \
      gettext \
      intl \
      ldap \
      mysqli \
      opcache \
      pcntl \
      pdo_mysql \
      shmop \
      soap \
      sockets \
      sysvmsg \
      sysvsem \
      sysvshm

RUN apt-get install -y \
        libzip-dev \
        zip \
  && docker-php-ext-install zip

RUN pecl install mcrypt-1.0.3 \
    docker-php-ext-enable mcrypt

#
# PHP configuration
#
# Set timezone
RUN echo "date.timezone = \"America/New_York\"" > $PHP_INI_DIR/conf.d/timezone.ini
# Increase PHP memory limit
RUN echo "memory_limit=-1" > $PHP_INI_DIR/conf.d/timezone.ini
# Set upload limit
RUN echo "upload_max_filesize = 128M\npost_max_size = 128M" > $PHP_INI_DIR/conf.d/00-max_filesize.ini


#
# Apache configuration
#
RUN a2enmod rewrite headers expires ssl \
  && sed -i "/User www-data/c\User \$\{APACHE_RUN_USER\}" /etc/apache2/apache2.conf \
  && sed -i "/Group www-data/c\Group \$\{APACHE_RUN_GROUP\}" /etc/apache2/apache2.conf \
  && sed -i "/DocumentRoot \/var\/www\/html/c\\\tDocumentRoot \$\{APACHE_DOCROOT\}" /etc/apache2/sites-enabled/000-default.conf \
  # Preemptively add a user 1000, for use with $APACHE_RUN_USER on osx
  && adduser --uid 1000 --gecos 'My OSX User' --disabled-password osxuser

RUN mkdir -p /root/work/
WORKDIR /root/work/

# install git
RUN apt-get -y update && apt-get -y install git && apt-get -y install lftp && apt-get -y install git-ftp

# slim down image
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/man/?? /usr/share/man/??_*

COPY test.sh /test.sh