ARG PHP_VERSION=7.4
FROM php:${PHP_VERSION}-alpine

RUN apk --update add wget \
    curl \
    git \
    build-base \
    libmcrypt-dev \
    libxml2-dev \
    pcre-dev \
    zlib-dev \
    autoconf \
    cyrus-sasl-dev \
    libgsasl-dev \
    oniguruma-dev \
    supervisor

RUN pecl channel-update pecl.php.net; \
    docker-php-ext-install mysqli mbstring pdo pdo_mysql tokenizer xml pcntl

RUN apk add --update --no-cache freetype-dev libjpeg-turbo-dev jpeg-dev libpng-dev; \
    docker-php-ext-configure gd --with-freetype --with-jpeg --with-png; \
    docker-php-ext-install gd

RUN set -eux; \
    apk --update add libzip-dev mysql-client ffmpeg && \
    docker-php-ext-configure zip; \
    docker-php-ext-install zip

RUN printf "\n" | pecl install -o -f redis; \
    rm -rf /tmp/pear; \
    docker-php-ext-enable redis

RUN if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
        pecl install xmlrpc-1.0.0RC2; \
        docker-php-ext-enable xmlrpc; \
    else \
        docker-php-ext-install xmlrpc; \
    fi;

COPY ./supervisor/supervisord.conf /etc/supervisor/supervisord.conf

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c",  "/etc/supervisor/supervisord.conf"]

RUN rm /var/cache/apk/* \
    && mkdir -p /var/www

WORKDIR /etc/supervisor/conf.d/
