###Starting

```
cp .env.example .env
install laravel
PATH_SOURCE=path_install_paravel
docker-compose up
```

##Mysql

```
docker build -t marcelocorrea/mysql:5.7 ./mysql
docker build -t marcelocorrea/mysql:8.0 ./mysql
docker push marcelocorrea/mysql:8.0
```

##PHP FPM

```
docker build -t marcelocorrea/php:8.0-fpm --build-arg PHP_VERSION=8.0 ./php-fpm
docker push marcelocorrea/php:8.0-fpm
docker build -t marcelocorrea/php:8.1-fpm --build-arg PHP_VERSION=8.1 ./php-fpm
docker push marcelocorrea/php:8.1-fpm

docker build -t marcelocorrea/php-fpm:7.4 --build-arg PHP_VERSION=7.4 ./php-fpm
docker push marcelocorrea/php-fpm:7.4
docker build -t marcelocorrea/php-fpm:8.1 --build-arg PHP_VERSION=8.1 ./php-fpm
docker push marcelocorrea/php-fpm:8.1
docker build -t marcelocorrea/php-fpm:8.2 --build-arg PHP_VERSION=8.2 ./php-fpm
docker push marcelocorrea/php-fpm:8.2
```

##PHP Worker

```
docker build -t marcelocorrea/php:8.0-worker --build-arg PHP_VERSION=8.0 ./php-worker
docker push marcelocorrea/php:8.0-worker

docker build -t marcelocorrea/php-worker:7.4 --build-arg PHP_VERSION=7.4 ./php-worker
docker push marcelocorrea/php-worker:7.4
docker build -t marcelocorrea/php-worker:8.1 --build-arg PHP_VERSION=8.1 ./php-worker
docker push marcelocorrea/php-worker:8.1

docker build -t marcelocorrea/php-worker:8.2 --build-arg PHP_VERSION=8.2  ./php-worker
docker push marcelocorrea/php-worker:8.2
```

##Nginx

```
docker build -t marcelocorrea/nginx:latest ./nginx
```