FROM php:8.1-apache

RUN apt-get update

ARG uid
ARG NODE_VERSION=16

WORKDIR /var/www/html

ENV APACHE_DOCUMENT_ROOT=/var/www/html/public

# Install development packages
RUN apt-get install -y \
    zip \
    curl \
    unzip \
    # supervisor \
    libzip-dev \
    libonig-dev \
    libicu-dev \
    libxml2-dev \
    libcurl4-openssl-dev \
    libpng-dev \
    libedit-dev \
    libc-client-dev \
    libkrb5-dev

RUN curl -sL https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g npm \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install -y yarn

RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Apache configs + document root
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf
COPY vhost.conf /etc/apache2/sites-available/000-default.conf

# Mod_rewrite for URL rewrite and mod_headers for .htaccess extra headers like Access-Control-Allow-Origin-
RUN a2enmod rewrite headers

# Start with base php config, then add extensions
RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"

# Install php extenstion
RUN docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-configure opcache --enable-opcache

RUN docker-php-ext-install \
    imap \
    intl \
    bcmath \
    opcache \
    mbstring \
    pdo_mysql \
    xml \
    curl \
    gd \
    zip

RUN pecl install -o -f redis \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# We need a user with the same UID/GID with host user
# so when we execute CLI commands, all the host file's ownership remains intact
# otherwise command from inside container will create root-owned files and directories
RUN useradd -G www-data,root -u $uid -d /home/sail sail
RUN mkdir -p /home/sail/.composer && \
    chown -R sail:sail /home/sail

# PHP config
COPY php/conf.d/opcache.ini /usr/local/etc/php/conf.d/99-opcache.ini
COPY php/conf.d/php.ini /usr/local/etc/php/conf.d/99-sail.ini

# Other
# COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
# CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
