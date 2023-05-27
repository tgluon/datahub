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
RUN  curl -sS https://repo1.maven.org/maven2/org/eclipse/jetty/jetty-runner/9.4.46.v20220331/jetty-runner-9.4.46.v20220331.jar --output jetty-runner.jar \
    && curl -sS https://repo1.maven.org/maven2/org/eclipse/jetty/jetty-jmx/9.4.46.v20220331/jetty-jmx-9.4.46.v20220331.jar --output jetty-jmx.jar \
    && curl -sS https://repo1.maven.org/maven2/org/eclipse/jetty/jetty-util/9.4.46.v20220331/jetty-util-9.4.46.v20220331.jar --output jetty-util.jar \
    && wget --no-verbose https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v1.24.0/opentelemetry-javaagent.jar \
    && wget --no-verbose https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/${JMX_VERSION}/jmx_prometheus_javaagent-${JMX_VERSION}.jar -O jmx_prometheus_javaagent.jar \
    && cp /usr/lib/jvm/java-11-openjdk/jre/lib/security/cacerts /tmp/kafka.client.truststore.jks

FROM base as prod-install
COPY datahub-upgrade.jar /datahub/datahub-upgrade/bin/
COPY metadata-models/src/main/resources/entity-registry.yml /datahub/datahub-gms/resources/entity-registry.yml

FROM base as dev-install
# Dummy stage for development. Assumes code is built on your machine and mounted to this image.
# See this excellent thread https://github.com/docker/cli/issues/1134

FROM ${APP_ENV}-install as final

ENV DOCKERIZE_VERSION v0.6.1
RUN mkdir -p /go/bin/ && chmod 755 /go/bin/
WORKDIR  /go/bin/
RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \

USER datahub

ENTRYPOINT ["java", "-jar", "/datahub/datahub-upgrade/bin/datahub-upgrade.jar"]
