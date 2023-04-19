ARG PHP_VERSION=7.4
FROM php:${PHP_VERSION}-alpine

RUN apk --update add wget \
  curl \
  git \
  build-base \
  libmcrypt-dev \
  libxml2-dev \
  linux-headers \
  pcre-dev \
  zlib-dev \
  autoconf \
  cyrus-sasl-dev \
  libgsasl-dev \
  oniguruma-dev \
  supervisor; \
  if [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "50600" ] || \
     [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "70000" ]; then \
    apk --update add libressl libressl-dev; \
  else \
    apk --update add openssl-dev; \
  fi

RUN pecl channel-update pecl.php.net; \
  docker-php-ext-install mysqli mbstring pdo pdo_mysql xml pcntl; \
  if [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "80100" ] || \
     [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "80200" ]; then \
    php -m | grep -oiE '^tokenizer$'; \
  else \
    docker-php-ext-install tokenizer; \
  fi

# Add a non-root user:
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

ARG INSTALL_GNUPG=true
RUN set -eux; if [ ${INSTALL_GNUPG} = true ]; then \
  apk add --no-cache --no-progress --virtual BUILD_DEPS_PHP_GNUPG gpgme-dev; \
  apk add --no-cache --no-progress gpgme; \
  pecl install gnupg; \
  docker-php-ext-enable gnupg; \
  fi

#Install GD package:
ARG INSTALL_GD=true
RUN if [ ${INSTALL_GD} = true ]; then \
  apk add --update --no-cache freetype-dev libjpeg-turbo-dev jpeg-dev libpng-dev; \
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "4" ]; then \
  docker-php-ext-configure gd --with-freetype --with-jpeg; \
  else \
  docker-php-ext-configure gd --with-freetype-dir=/usr/lib/ --with-jpeg-dir=/usr/lib/ --with-png-dir=/usr/lib/; \
  fi; \
  docker-php-ext-install gd \
  ;fi

ARG INSTALL_BCMATH=true
RUN if [ ${INSTALL_BCMATH} = true ]; then \
  docker-php-ext-install bcmath \
  ;fi

ARG INSTALL_ZIP_ARCHIVE=true
RUN set -eux; \
  if [ ${INSTALL_ZIP_ARCHIVE} = true ]; then \
  apk --update add libzip-dev && \
  if [ ${PHP_VERSION} = "7.3" ] || [ ${PHP_VERSION} = "7.4" ] || [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
  docker-php-ext-configure zip; \
  else \
  docker-php-ext-configure zip --with-libzip; \
  fi && \
  # Install the zip extension
  docker-php-ext-install zip \
  ;fi

ARG INSTALL_MYSQL_CLIENT=true
RUN if [ ${INSTALL_MYSQL_CLIENT} = true ]; then \
  apk --update add mysql-client \
  ;fi

ARG INSTALL_FFMPEG=true
RUN if [ ${INSTALL_FFMPEG} = true ]; then \
  apk --update add ffmpeg \
  ;fi

ARG INSTALL_REDIS=true
RUN if [ ${INSTALL_REDIS} = true ]; then \
  # Install Redis Extension
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
  printf "\n" | pecl install -o -f redis-4.3.0; \
  else \
  printf "\n" | pecl install -o -f redis; \
  fi; \
  rm -rf /tmp/pear; \
  docker-php-ext-enable redis \
  ;fi

ARG INSTALL_XMLRPC=true

RUN if [ ${INSTALL_XMLRPC} = true ]; then \
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
  pecl install xmlrpc-1.0.0RC3; \
  docker-php-ext-enable xmlrpc; \
  else \
  docker-php-ext-install xmlrpc; \
  fi; \
  php -m | grep -r 'xmlrpc'; \
  fi

USER root

ARG INSTALL_INTL=false

RUN if [ ${INSTALL_INTL} = true ]; then \
    apk add icu-dev && \
    docker-php-ext-configure intl && \
    docker-php-ext-install intl && \
    docker-php-ext-enable intl \
;fi

USER root

COPY ./supervisor/supervisord.conf /etc/supervisor/supervisord.conf

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c",  "/etc/supervisor/supervisord.conf"]

RUN php -v | head -n 1 | grep -q "PHP ${PHP_VERSION}."

RUN rm /var/cache/apk/* \
  && mkdir -p /var/www

WORKDIR /etc/supervisor/conf.d/