FROM golang:1-alpine3.17 AS binary

ENV DOCKERIZE_VERSION v0.6.1
WORKDIR /go/src/github.com/jwilder

RUN apk --no-cache --update add openssl git tar curl wget

WORKDIR /go/src/github.com/jwilder/dockerize



FROM centos AS base

# PFP-260: Upgrade Sqlite to >=3.28.0-r0 to fix https://security.snyk.io/vuln/SNYK-ALPINE39-SQLITE-449762
RUN  yum install mysql-client bash mariadb-connector-c sqlite wget

ENV DOCKERIZE_VERSION v0.6.1
RUN mkdir -p /go/bin/ && chmod 755 /go/bin/
WORKDIR  /go/bin/

RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \

COPY docker/mysql-setup/init.sql /init.sql
COPY docker/mysql-setup/init.sh /init.sh
RUN chmod 755 init.sh

ENV DATAHUB_DB_NAME="datahub"

CMD dockerize -wait tcp://$MYSQL_HOST:$MYSQL_PORT -timeout 240s /init.sh
