# Defining environment
ARG APP_ENV=prod

FROM centos:7.9.2009 AS base

RUN export LANG=zh_CN.UTF-8
RUN rm -f /etc/localtime && ln -sv /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone

# 创建用户和用户组，跟yaml编排里的spec.template.spec.containers. securityContext.runAsUser: 9999
RUN groupadd --system --gid=9999 datahub && useradd --system --home-dir /home/datahub --uid=9999 --gid=datahub datahub

# 安装sudo
RUN yum -y install sudo curl coreutils gcompat mysql-client mariadb-connector-c sqlite wget java-11-openjdk java-11-openjdk-devel

RUN chmod 640 /etc/sudoers

ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.12.0.7-0.el7_9.x86_64
ENV JRE_HOME=$JAVA_HOME/jre
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/jre/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH=$JAVA_HOME/bin:$PATH

# Upgrade Alpine and base packages
ENV JMX_VERSION=0.18.0
# PFP-260: Upgrade Sqlite to >=3.28.0-r0 to fix https://security.snyk.io/vuln/SNYK-ALPINE39-SQLITE-449762
RUN wget --no-verbose https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v1.24.0/opentelemetry-javaagent.jar \
    && wget --no-verbose https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/${JMX_VERSION}/jmx_prometheus_javaagent-${JMX_VERSION}.jar -O jmx_prometheus_javaagent.jar \
    && cp /usr/lib/jvm/java-11-openjdk/jre/lib/security/cacerts /tmp/kafka.client.truststore.jks


FROM base as prod-install
COPY mce-consumer-job.jar /datahub/datahub-mce-consumer/bin/
COPY metadata-models/src/main/resources/entity-registry.yml /datahub/datahub-mce-consumer/resources/entity-registry.yml
COPY docker/datahub-mce-consumer/start.sh /datahub/datahub-mce-consumer/scripts/
COPY docker/monitoring/client-prometheus-config.yaml /datahub/datahub-mce-consumer/scripts/prometheus-config.yaml
RUN chmod +x /datahub/datahub-mce-consumer/scripts/start.sh

FROM base as dev-install
# Dummy stage for development. Assumes code is built on your machine and mounted to this image.
# See this excellent thread https://github.com/docker/cli/issues/1134
COPY metadata-models/src/main/resources/entity-registry.yml /datahub/datahub-mce-consumer/resources/entity-registry.yml

FROM ${APP_ENV}-install as final

ENV DOCKERIZE_VERSION v0.6.1
RUN mkdir -p /go/bin/ && chmod 755 /go/bin/
WORKDIR  /go/bin/

RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \

USER datahub

ENV JMX_OPTS=""
ENV JAVA_OPTS=""

EXPOSE 9090

HEALTHCHECK --start-period=2m --retries=4 CMD curl --fail http://localhost:9090/actuator/health || exit 1

CMD /datahub/datahub-mce-consumer/scripts/start.sh
