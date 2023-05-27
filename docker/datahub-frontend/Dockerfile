# Defining environment
ARG APP_ENV=prod

FROM centos:7.9.2009 AS base

RUN export LANG=zh_CN.UTF-8
RUN rm -f /etc/localtime && ln -sv /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone

# 创建用户和用户组，跟yaml编排里的spec.template.spec.containers. securityContext.runAsUser: 9999
RUN groupadd --system --gid=9999 datahub && useradd --system --home-dir /home/datahub --uid=9999 --gid=datahub datahub

# 安装sudo
RUN yum -y install sudo mysql-client mariadb-connector-c sqlite wget java-11-openjdk java-11-openjdk-devel

RUN chmod 640 /etc/sudoers

ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.12.0.7-0.el7_9.x86_64
ENV JRE_HOME=$JAVA_HOME/jre
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/jre/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH=$JAVA_HOME/bin:$PATH

FROM base as prod-install

COPY ./datahub-frontend.zip /
RUN unzip datahub-frontend.zip && rm datahub-frontend.zip
COPY ./docker/monitoring/client-prometheus-config.yaml /datahub-frontend/
RUN chown -R datahub:datahub /datahub-frontend && chmod 755 /datahub-frontend

ENV JMX_VERSION=0.18.0
RUN wget https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v1.24.0/opentelemetry-javaagent.jar -O opentelemetry-javaagent.jar \
    && wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/${JMX_VERSION}/jmx_prometheus_javaagent-${JMX_VERSION}.jar -O jmx_prometheus_javaagent.jar

FROM base as dev-install
# Dummy stage for development. Assumes code is built on your machine and mounted to this image.
# See this excellent thread https://github.com/docker/cli/issues/1134
VOLUME [ "/datahub-frontend" ]

FROM ${APP_ENV}-install as final
USER datahub

ARG SERVER_PORT=9002
ENV SERVER_PORT=$SERVER_PORT
RUN echo $SERVER_PORT
EXPOSE $SERVER_PORT

HEALTHCHECK --start-period=2m --retries=4 CMD curl --fail http://localhost:$SERVER_PORT/admin || exit 1
COPY ./docker/datahub-frontend/start.sh /
CMD ./start.sh
