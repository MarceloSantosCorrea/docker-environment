###Starting

```
copy .env.example .env
docker-compose up
```

##Mysql

```
docker build -t marcelocorrea/mysql:5.7 ./mysql
```

##PHP FPM

```
docker build -t marcelocorrea/php7.4-fpm ./php-fpm
docker build -t marcelocorrea/php8.0-fpm ./php-fpm
```

##PHP Worker

```
docker build -t marcelocorrea/php7.4-worker ./php-worker
docker build -t marcelocorrea/php8.0-worker ./php-worker
```

##Nginx

```
docker build -t marcelocorrea/nginx:latest ./nginx
```