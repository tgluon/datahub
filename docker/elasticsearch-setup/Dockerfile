# This "container" is a workaround to pre-create search indices

# Defining environment
ARG APP_ENV=prod

FROM centos:7.9.2009 AS base

RUN export LANG=zh_CN.UTF-8
RUN rm -f /etc/localtime && ln -sv /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone

# 创建用户和用户组，跟yaml编排里的spec.template.spec.containers. securityContext.runAsUser: 9999
RUN groupadd --system --gid=9999 datahub && useradd --system --home-dir /home/datahub --uid=9999 --gid=datahub datahub

# 安装sudo
RUN yum -y install sudo gcc git curl libmpfr4 openssl openssl-devel coreutils gcompat mysql-client mariadb-connector-c sqlite wget java-11-openjdk java-11-openjdk-devel

RUN chmod 640 /etc/sudoers

ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.12.0.7-0.el7_9.x86_64
ENV JRE_HOME=$JAVA_HOME/jre
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/jre/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH=$JAVA_HOME/bin:$PATH


ENV GO_VERSION 1.20.4
ENV GO_SHA256SUM 979694c2c25c735755bf26f4f45e19e64e4811d661dd07b8c010f7a8e18adfca
RUN curl -L https://golang.org/dl/go$GO_VERSION.linux-amd64.tar.gz -o go$GO_VERSION.linux-amd64.tar.gz
RUN echo "$GO_SHA256SUM go$GO_VERSION.linux-amd64.tar.gz" | sha256sum -c
RUN tar -C /usr/local -xzf go$GO_VERSION.linux-amd64.tar.gz && rm -f go$GO_VERSION.linux-amd64.tar.gz
ENV PATH /usr/local/go/bin:$PATH
ENV GOROOT /usr/local/go
RUN go env -w GOPROXY="https://goproxy.cn,direct"

ENV DOCKERIZE_VERSION v0.6.1
RUN mkdir -p /go/bin/ && chmod 755 /go/bin/
WORKDIR  /go/bin/

RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \

COPY docker/elasticsearch-setup/create-indices.sh /
RUN chmod 755 create-indices.sh
COPY metadata-service/restli-servlet-impl/src/main/resources/index /index

CMD if [ "$ELASTICSEARCH_USE_SSL" == "true" ]; then ELASTICSEARCH_PROTOCOL=https; else ELASTICSEARCH_PROTOCOL=http; fi \
    && if [[ -n "$ELASTICSEARCH_USERNAME" ]]; then ELASTICSEARCH_HTTP_HEADERS="Authorization: Basic $(echo -ne "$ELASTICSEARCH_USERNAME:$ELASTICSEARCH_PASSWORD" | base64)"; else ELASTICSEARCH_HTTP_HEADERS="Accept: */*"; fi \
    && if [[ "$SKIP_ELASTICSEARCH_CHECK" != "true" ]]; then \
        dockerize -wait $ELASTICSEARCH_PROTOCOL://$ELASTICSEARCH_HOST:$ELASTICSEARCH_PORT -wait-http-header "${ELASTICSEARCH_HTTP_HEADERS}" -timeout 120s /create-indices.sh; \
    else /create-indices.sh; fi

