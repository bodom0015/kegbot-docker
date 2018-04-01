# Kegbot on Docker

Docker support for Kegbot.  

[![CircleCI](https://circleci.com/gh/Kegbot/kegbot-docker.svg?style=svg)](https://circleci.com/gh/Kegbot/kegbot-docker)

## Quick start

 Requirements:
* [Docker](https://docs.docker.com/engine/installation/) 1.12+
* [Docker-compose](https://docs.docker.com/compose/install/) 1.9+

To start Kegbot:
```bash
$ docker-compose up
```

To stop Kegbot:

```bash
$ docker-compose down
```

Data is saved in `~/tmp/kegbot` folder on the local disk to prevent data loss during restarts.
You can create a symbolic link should you choose to save the data somewhere else.


## Without Compose
Build necessary images:
```bash
docker build -t kegbot/nginx -f nginx/Dockerfile nginx
docker build -t kegbot/server -f server/Dockerfile server
```

Run containers from newly built images:
```bash
# Run dependencies ("redis" / "db")
docker run -it -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=kegbot-db-pass --name=kegbot-db  kegbot/mysql
docker run -it -d -p 6379:6379 --name=kegbot-redis redis:3-alpine
```

Once MySQL comes up, you'll need to shell in and add a new user:
```bash
docker exec -it kegbot-db bash
mysql --password=kegbot-db-pass
CREATE USER 'kegbot'@'localhost' IDENTIFIED BY 'kegbot-db-pass';
CREATE USER 'kegbot'@'%' IDENTIFIED BY 'kegbot-db-pass';
GRANT ALL ON *.* TO 'kegbot'@'localhost';
GRANT ALL ON *.* TO 'kegbot'@'%';
CREATE DATABASE kegbot;
exit
```

Test that `kegbot` database user can access MySQL:
```bash
mysql -u kegbot --pass=kegbot-db-pass
SELECT host,user from mysql.user;
```

Finally, run the KegBot application:
```bash
# Run Python API server ("server")
docker run -it -d -e KEGBOT_DEBUG=true -e KEGBOT_DB_HOST=kegbot-db -e KEGBOT_REDIS_HOST=kegbot-redis -e KEGBOT_REDIS_PORT=6379 -e KEGBOT_DB_PORT=3306 -e KEGBOT_DB_USER=kegbot -e KEGBOT_DB_NAME=kegbot -e KEGBOT_SETTINGS_DIR=/etc/kegbot -e KEGBOT_DB_PASS=kegbot-db-pass -v c:/Users/child/kegbot-tmp/server:/kegbot-data --link=kegbot-db --link=kegbot-redis --name=kegbot-server kegbot/server

# Run web frontend ("nginx")
docker run -it -d -v c:/Users/child/kegbot-tmp/server:/kegbot-data:ro --link=kegbot-server:kegbot.docker -p 80:80 --name=kegbot-nginx kegbot/nginx
```

## Credit

Thanks to [@blalor/docker-kegbot-server](https://github.com/blalor/docker-kegbot-server)
and [@dencold/kegbot](https://github.com/dencold/kegbot) for inspiring this
project.
