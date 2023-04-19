ARG PHP_VERSION=7.4
FROM php:${PHP_VERSION}-fpm

RUN set -eux; \
apt-get update; \
apt-get upgrade -y; \
apt-get install -y build-essential libcurl3-dev libcurl4 openssl libssl-dev libssh-dev zlib1g-dev zlib1g libbrotli-dev brotli libkrb5-dev libldap-dev librtmp-dev libpsl-dev libnghttp2-dev zstd libzstd-dev libssh2-1 libssh2-1-dev

RUN set -eux; \
apt-get update; \
apt-get upgrade -y; \
apt-get install -y --no-install-recommends \
git \
curl \
libmemcached-dev \
libz-dev \
libpq-dev \
libjpeg-dev \
libpng-dev \
libfreetype6-dev \
libwebp-dev \
libmcrypt-dev \
libonig-dev; \
rm -rf /var/lib/apt/lists/*

RUN git clone --recursive --depth=1 https://github.com/kjdev/php-ext-zstd.git && \
cd php-ext-zstd && \
phpize && \
./configure && \
make && \
make install && \
./configure --with-libzstd && \
pecl install zstd && \
docker-php-ext-enable zstd

RUN set -eux; \
docker-php-ext-install pdo_mysql; \
docker-php-ext-configure gd \
--prefix=/usr \
--with-jpeg \
--with-webp \
--with-freetype; \
docker-php-ext-install gd;

RUN set -xe; \
apt-get update -yqq && \
pecl channel-update pecl.php.net && \
apt-get install -yqq \
apt-utils \
gnupg2 \
git \
libzip-dev zip unzip && \
docker-php-ext-configure zip; \
docker-php-ext-install zip

RUN apt-get -y install libssh2-1-dev && \
pecl install -a ssh2-1.3.1; \
docker-php-ext-enable ssh2

RUN pecl install -o -f redis; \
rm -rf /tmp/pear \
&& docker-php-ext-enable redis

RUN docker-php-ext-install pcntl

RUN apt-get install -yqq zlib1g-dev libicu-dev g++ && \
docker-php-ext-configure intl && \
docker-php-ext-install intl

RUN apt-get install -yqq libmagickwand-dev imagemagick && \
cd /tmp && \
git clone https://github.com/Imagick/imagick; \
cd imagick && \
phpize && \
./configure && \
make && \
make install && \
rm -r /tmp/imagick; \
docker-php-ext-enable imagick;

RUN apt-get install -yqq librdkafka-dev && \
pecl install rdkafka && \
docker-php-ext-enable rdkafka

RUN apt-get -y install default-mysql-client \
ffmpeg

RUN apt-get -yq install libxml2-dev; \
if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
  pecl install xmlrpc-1.0.0RC3; \
  docker-php-ext-enable xmlrpc; \
else \
  docker-php-ext-install xmlrpc; \
fi

RUN docker-php-ext-install exif bcmath

#RUN apt-get install -y wget && \
#cd /usr/local/src && \
#rm -rf curl* && \
#wget https://curl.haxx.se/download/curl-7.73.0.tar.gz && \
#tar -xvf curl-7.73.0.tar.gz && \
#cd curl-7.73.0 && \
#./configure --with-libzstd --with-libssh2 --with-libmetalink --with-ngtcp2 && \
#make && \
#make install && \
#cp /usr/local/bin/curl /usr/bin/curl && \
#ldconfig

###########################################################################
# xDebug:
###########################################################################

ARG XDEBUG_PORT=9003

RUN if [ ${PHP_VERSION} = "8.2.5" ]; then \
  pecl install xdebug-3.2.1; \
else \
  pecl install xdebug-3.1.4; \
fi && \
docker-php-ext-enable xdebug

# Copy xdebug configuration for remote debugging
COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

RUN if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ] || { [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && { [ $(php -r "echo PHP_MINOR_VERSION;") = "4" ] || [ $(php -r "echo PHP_MINOR_VERSION;") = "3" ] ;} ;}; then \
  sed -i "s/xdebug.remote_host=/xdebug.client_host=/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.remote_connect_back=0/xdebug.discover_client_host=false/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.remote_port=9000/xdebug.client_port=${XDEBUG_PORT}/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.profiler_enable=0/; xdebug.profiler_enable=0/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.profiler_output_dir=/xdebug.output_dir=/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.remote_mode=req/; xdebug.remote_mode=req/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.remote_autostart=0/xdebug.start_with_request=yes/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.remote_enable=0/xdebug.mode=debug/" /usr/local/etc/php/conf.d/xdebug.ini \
;else \
  sed -i "s/xdebug.remote_autostart=0/xdebug.remote_autostart=1/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.remote_enable=0/xdebug.remote_enable=1/" /usr/local/etc/php/conf.d/xdebug.ini \
;fi
RUN sed -i "s/xdebug.cli_color=0/xdebug.cli_color=1/" /usr/local/etc/php/conf.d/xdebug.ini

COPY ./laravel.ini /usr/local/etc/php/conf.d
COPY ./xlaravel.pool.conf /usr/local/etc/php-fpm.d/

USER root

# Clean up
RUN apt-get clean && \
rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
rm /var/log/lastlog /var/log/faillog

# Configure non-root user.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN groupmod -o -g ${PGID} www-data && \
usermod -o -u ${PUID} -g www-data www-data

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www

RUN chown www-data:www-data -R /var/www
RUN chmod 775 -R /var/www

USER www-data

CMD ["php-fpm"]

EXPOSE 9000
