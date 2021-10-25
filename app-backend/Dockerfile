########################################################################
### Composer

FROM composer:1.10 AS composer

COPY composer.json /app
COPY composer.lock /app

RUN composer install        \
    --ignore-platform-reqs  \
    --no-ansi               \
    --no-autoloader         \
    --no-interaction        \
    --no-scripts

COPY . /app/
RUN composer dump-autoload --optimize --classmap-authoritative

### Composer
########################################################################
### NodeJS

FROM node:14.17.0-alpine3.12 AS node

WORKDIR /app

COPY package.json           /app
COPY package-lock.json      /app
COPY webpack.mix.js         /app
COPY /resources             /app/resources

RUN npm install && npm run dev

### NodeJS
########################################################################
### PHP

FROM php:7.4-apache

RUN apt-get update

# 1. development packages
RUN apt-get install -y \
    git \
    zip \
    curl \
    sudo \
    unzip \
    libicu-dev \
    libbz2-dev \
    libpng-dev \
    libjpeg-dev \
    libmcrypt-dev \
    libreadline-dev \
    libfreetype6-dev \
    g++

# 2. apache configs + document root
ENV APACHE_DOCUMENT_ROOT=/var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

# 3. mod_rewrite for URL rewrite and mod_headers for .htaccess extra headers like Access-Control-Allow-Origin-
RUN a2enmod rewrite headers

# 4. start with base php config, then add extensions
RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"

RUN docker-php-ext-install mysqli pdo pdo_mysql

# 5. composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

COPY . /var/www
COPY --from=composer /app/vendor                /var/www/html/vendor
COPY --from=node     /app/public                /var/www/html/public/
COPY --from=node     /app/mix-manifest.json     /var/www/html/mix-manifest.json

COPY --chown=www:www-data . /var/www

WORKDIR /var/www

### PHP
########################################################################
