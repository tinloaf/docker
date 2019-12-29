# What is wallabag?

[![Build Status](https://travis-ci.org/wallabag/docker.svg?branch=master)](https://travis-ci.org/wallabag/docker)
[![Docker Stars](https://img.shields.io/docker/stars/wallabag/wallabag.svg?maxAge=2592000)](https://hub.docker.com/r/wallabag/wallabag/)
[![Docker Pulls](https://img.shields.io/docker/pulls/wallabag/wallabag.svg?maxAge=2592000)](https://hub.docker.com/r/wallabag/wallabag/)

[wallabag](https://www.wallabag.org/) is a self hostable application for saving web pages. Unlike other services, wallabag is free (as in freedom) and open source.

With this application you will not miss content anymore. Click, save, read it when you want. It saves the content you select so that you can read it when you have time.

# How to use this image

Default login is `wallabag:wallabag`.

## Environment variables

- `-e MYSQL_ROOT_PASSWORD=...` (needed for the mariadb container to initialise and for the entrypoint in the wallabag container to create a database and user if its not there)
- `-e POSTGRES_PASSWORD=...` (needed for the postgres container to initialise and for the entrypoint in the wallabag container to create a database and user if not there)
- `-e POSTGRES_USER=...` (needed for the posgres container to initialise and for the entrypoint in the wallabag container to create a database and user if not there)
- `-e SYMFONY__ENV__DATABASE_DRIVER=...` (defaults to "pdo_sqlite", this sets the database driver to use)
- `-e SYMFONY__ENV__DATABASE_DRIVER_CLASS=...` (sets the database driver class to use)
- `-e SYMFONY__ENV__DATABASE_HOST=...` (defaults to "127.0.0.1", if use mysql this should be the name of the mariadb container)
- `-e SYMFONY__ENV__DATABASE_PORT=...` (port of the database host)
- `-e SYMFONY__ENV__DATABASE_NAME=...`(defaults to "symfony", this is the name of the database to use)
- `-e SYMFONY__ENV__DATABASE_USER=...` (defaults to "root", this is the name of the database user to use)
- `-e SYMFONY__ENV__DATABASE_PASSWORD=...` (defaults to "~", this is the password of the database user to use)
- `-e SYMFONY__ENV__DATABASE_CHARSET=...` (defaults to utf8, this is the database charset to use)
- `-e SYMFONY__ENV__SECRET=...` (defaults to "ovmpmAWXRCabNlMgzlzFXDYmCFfzGv")
- `-e SYMFONY__ENV__LOCALE=...` (default to en)
- `-e SYMFONY__ENV__MAILER_HOST=...`  (defaults to "127.0.0.1", the SMTP host)
- `-e SYMFONY__ENV__MAILER_USER=...` (defaults to "~", the SMTP user)
- `-e SYMFONY__ENV__MAILER_PASSWORD=...`(defaults to "~", the SMTP password)
- `-e SYMFONY__ENV__FROM_EMAIL=...`(defaults to "`wallabag@example.com`", the address wallabag uses for outgoing emails)
- `-e SYMFONY__ENV__FOSUSER_REGISTRATION=...`(defaults to "true", enable or disable public user registration)
- `-e SYMFONY__ENV__FOSUSER_CONFIRMATION=...`(defaults to "true", enable or disable registration confirmation)
- `-e SYMFONY__ENV__DOMAIN_NAME=...`  defaults to "`https://your-wallabag-url-instance.com`", the URL of your wallabag instance)
- `-e SYMFONY__ENV__REDIS_SCHEME=...` (defaults to "tcp", protocol to use to communicate with the target server (tcp, unix, or http))
- `-e SYMFONY__ENV__REDIS_HOST=...` (defaults to "redis", IP or hostname of the target server)
- `-e SYMFONY__ENV__REDIS_PORT=...` (defaults to "6379", port of the target host)
- `-e SYMFONY__ENV__REDIS_PATH=...`(defaults to "~", path of the unix socket file)
- `-e SYMFONY__ENV__REDIS_PASSWORD=...` (defaults to "~", this is the password defined in the Redis server configuration)
- `-e SYMFONY__ENV__SENTRY_DSN=...` (defaults to "~", this is the data source name for sentry)
- `-e POPULATE_DATABASE=...`(defaults to "True". Does the DB has to be populated or is it an existing one)

## SQLite

The easiest way to start wallabag is to use the SQLite backend. You can spin that up with

```
$ docker run -p 80:80 wallabag/wallabag
```

and point your browser to `http://localhost:80`. For persistent storage you should start the container with a volume:

```
$ docker run -v /opt/wallabag/data:/var/www/wallabag/data -v /opt/wallabag/images:/var/www/wallabag/web/assets/images -p 80:80 wallabag/wallabag
```

## MariaDB / MySQL

For using MariaDB or MySQL you have to define some environment variables with the container. Example:

```
$ docker run --name wallabag-db -e "MYSQL_ROOT_PASSWORD=my-secret-pw" -d mariadb
$ docker run --name wallabag --link wallabag-db:wallabag-db -e "MYSQL_ROOT_PASSWORD=my-secret-pw" -e "SYMFONY__ENV__DATABASE_DRIVER=pdo_mysql" -e "SYMFONY__ENV__DATABASE_HOST=wallabag-db" -e "SYMFONY__ENV__DATABASE_PORT=3306" -e "SYMFONY__ENV__DATABASE_NAME=wallabag" -e "SYMFONY__ENV__DATABASE_USER=wallabag" -e "SYMFONY__ENV__DATABASE_PASSWORD=wallapass" -e "SYMFONY__ENV__DATABASE_CHARSET=utf8mb4" -p 80:80 wallabag/wallabag
```

## PostgreSQL

For using PostgreSQL you have to define some environment variables with the container. Example:

```
$ docker run --name wallabag-db -e "POSTGRES_PASSWORD=my-secret-pw" -e "POSTGRES_USER=my-super-user" -d postgres:9.6
$ docker run --name wallabag --link wallabag-db:wallabag-db -e "POSTGRES_PASSWORD=my-secret-pw" -e "POSTGRES_USER=my-super-user" -e "SYMFONY__ENV__DATABASE_DRIVER=pdo_pgsql" -e 'SYMFONY__ENV__DATABASE_DRIVER_CLASS=Wallabag\CoreBundle\Doctrine\DBAL\Driver\CustomPostgreSQLDriver' -e "SYMFONY__ENV__DATABASE_HOST=wallabag-db" -e "SYMFONY__ENV__DATABASE_PORT=5432" -e "SYMFONY__ENV__DATABASE_NAME=wallabag" -e "SYMFONY__ENV__DATABASE_USER=wallabag" -e "SYMFONY__ENV__DATABASE_PASSWORD=wallapass" -p 80:80 wallabag/wallabag
```

## Redis

To use redis with a Docker link, a redis container with the name `redis` is needed and none of the `REDIS` environmental variables are needed:

 ```
$ docker run -p 6379:6379 --name redis redis:alpine
$ docker run -p 80:80 --link redis:redis wallabag/wallabag
```

To use redis with an external redis host, set the appropriate environmental variables. Example:

```
$ docker run -p 80:80 -e "SYMFONY__ENV__REDIS_HOST=my.server.hostname" -e "SYMFONY__ENV__REDIS_PASSWORD=my-secret-pw" wallabag/wallabag
```

## Upgrading

If there is a version upgrade that needs a database migration. The most easy way to do is running the `migrate` command:

```
$ docker run --link wallabag-db:wallabag-db -e <... your config variables here ...>  wallabag/wallabag migrate
```

Or you can start the container with the new image and run the migration command manually:

```
$ docker exec -t NAME_OR_ID_OF_YOUR_WALLABAG_CONTAINER /var/www/wallabag/bin/console doctrine:migrations:migrate --env=prod --no-interaction
```

## docker-compose

It's a good way to use [docker-compose](https://docs.docker.com/compose/). Example:

```
version: '3'
services:
  wallabag:
    image: wallabag/wallabag
    environment:
      - MYSQL_ROOT_PASSWORD=wallaroot
      - SYMFONY__ENV__DATABASE_DRIVER=pdo_mysql
      - SYMFONY__ENV__DATABASE_HOST=db
      - SYMFONY__ENV__DATABASE_PORT=3306
      - SYMFONY__ENV__DATABASE_NAME=wallabag
      - SYMFONY__ENV__DATABASE_USER=wallabag
      - SYMFONY__ENV__DATABASE_PASSWORD=wallapass
      - SYMFONY__ENV__DATABASE_CHARSET=utf8mb4
      - SYMFONY__ENV__MAILER_HOST=127.0.0.1
      - SYMFONY__ENV__MAILER_USER=~
      - SYMFONY__ENV__MAILER_PASSWORD=~
      - SYMFONY__ENV__FROM_EMAIL=wallabag@example.com
      - SYMFONY__ENV__DOMAIN_NAME=https://your-wallabag-url-instance.com
    ports:
      - "80"
    volumes:
      - /opt/wallabag/images:/var/www/wallabag/web/assets/images
      - /opt/wallabag/data:/var/www/wallabag/data
  db:
    image: mariadb
    environment:
      - MYSQL_ROOT_PASSWORD=wallaroot
    volumes:
      - /opt/wallabag/data:/var/lib/mysql
  redis:
    image: redis:alpine
```

Note that you must fill out the mail related variables according to your mail config. Also, you must create the directories `/opt/wallabag/images` and `/opt/wallabag/data` on the host.

## nginx

I use nginx to make wallabag public available. This is a example how to use it:

```
server {
        listen 443;
        server_name wallabag.foo.bar;

	ssl on;
        ssl_certificate /etc/letsencrypt/live/wallabag.foo.bar/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/wallabag.foo.bar/privkey.pem;

	location / {
		proxy_pass http://wallabag;
		proxy_set_header X-Forwarded-Host $server_name;
                proxy_set_header X-Forwarded-Proto https;
                proxy_set_header X-Forwarded-For $remote_addr;
	}

}
```

## Import worker

To run the [async redis import worker](https://doc.wallabag.org/en/admin/asynchronous.html#install-redis-for-asynchronous-tasks) use the following command:
```
$ docker run --name wallabag --link wallabag-db:wallabag-db --link redis:redis -e <... your config variables here ...>  wallabag/wallabag import <type>
```
Where `<type>` is one of pocket, readability, instapaper, wallabag_v1, wallabag_v2, firefox or chrome.
