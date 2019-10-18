## ARG JAVA_IMAGE=openjdk
## ARG JAVA_TAG=8-alpine
## FROM ${JAVA_IMAGE}:${JAVA_TAG}
FROM openjdk:11.0.3-jdk-slim-stretch

ARG GS_PRODUCT=insightedge
ARG GS_VERSION=15.0.0
ARG GS_MILESTONE=m15
ARG GS_BUILD=20700-87
ARG GS_BUILD_NAME=15.0.0-m15
ARG GS_NAME=gigaspaces-${GS_PRODUCT}-enterprise-${GS_BUILD_NAME}
ARG GS_URL=https://gigaspaces-releases-eu.s3.amazonaws.com/${GS_PRODUCT}/${GS_VERSION}/${GS_NAME}.zip
ENV GS_HOME /opt/gigaspaces
WORKDIR ${GS_HOME}
# Download $GS_URL and unzip to $GS_HOME
## RUN apk --no-cache add openssl wget unzip &&\
RUN apt-get update --yes &&\
    apt-get install --yes openssl wget unzip &&\
    wget --progress=bar:force -O /tmp/gigaspaces.zip ${GS_URL} &&\
    unzip -q /tmp/gigaspaces.zip -d /tmp &&\
    mkdir -p ${GS_HOME} &&\
    mv /tmp/${GS_NAME}/* ${GS_HOME}/ &&\
    rm -f /tmp/gigaspaces.zip

## tini is used in the docker-entrypoint.sh
ENV TINI_VERSION v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /sbin/tini
RUN chmod +x /sbin/tini

## RUN apk add --no-cache nss
## RUN apk update && apk add --no-cache libc6-compat
# Configure and expose ports
ENV LD_LIBRARY_PATH "/lib:/lib64"
ENV GS_NIC_ADDRESS "#eth0:ip#"
ENV GS_LOOKUP_PORT 4174
ENV GS_LRMI_PORT 8200-8300
ENV GS_MANAGER_REST_PORT 8090
ENV GS_WEBUI_PORT 8099
ENV GS_WEBSTER_HTTP_PORT 8199
ENV GS_RMI_REGISTRY_PORT 10098
ENV GS_REST_PORT 8089
ENV GS_ZOOKEEPER_CLIENT_PORT 2181
ENV GS_MANAGER_ZOOKEEPER_DISCOVERY_PORT 2888
ENV GS_MANAGER_ZOOKEEPER_LEADER_ELECTION_PORT 3888
ENV SPARK_MASTER_PORT 7077
ENV SPARK_MASTER_WEBUI_PORT 8080
ENV SPARK_MASTER_REST_PORT 6066
ENV ZEPPELIN_PORT 9090
EXPOSE ${GS_LOOKUP_PORT} ${GS_LRMI_PORT} ${GS_MANAGER_REST_PORT} ${GS_WEBUI_PORT} ${GS_WEBSTER_HTTP_PORT} ${GS_RMI_REGISTRY_PORT} ${GS_REST_PORT} ${GS_ZOOKEEPER_CLIENT_PORT} ${GS_MANAGER_ZOOKEEPER_DISCOVERY_PORT} ${GS_MANAGER_ZOOKEEPER_LEADER_ELECTION_PORT} ${SPARK_MASTER_PORT} ${SPARK_MASTER_WEBUI_PORT} ${SPARK_MASTER_REST_PORT} ${ZEPPELIN_PORT}
ENV GS_OPTIONS_EXT "-Dcom.gs.multicast.discoveryPort=${GS_LOOKUP_PORT} -Dcom.gs.transport_protocol.lrmi.bind-port=${GS_LRMI_PORT} -Dcom.gs.manager.rest.port=${GS_MANAGER_REST_PORT} -Dorg.openspaces.launcher.port=${GS_WEBUI_PORT} -Dcom.gigaspaces.start.httpPort=${GS_WEBSTER_HTTP_PORT} -Dcom.gigaspaces.start.rmi.registryPort=${GS_RMI_REGISTRY_PORT} -Dcom.gs.manager.zookeeper.discovery.port=${GS_MANAGER_ZOOKEEPER_DISCOVERY_PORT} -Dcom.gs.manager.zookeeper.leader-election.port=${GS_MANAGER_ZOOKEEPER_LEADER_ELECTION_PORT} -Dzeppelin.server.port=${ZEPPELIN_PORT}"

# Configure Spark directory structure
ENV SPARK_HOME ${GS_HOME}/insightedge/spark
ENV SPARK_CONF_DIR "${SPARK_HOME}/conf"
## Below is from spark Docker file
## RUN set -ex && \
##     apk upgrade --no-cache && \
##     apk add --no-cache bash tini libc6-compat linux-pam && \
RUN set -ex && \
    mkdir -p ${SPARK_HOME} && \
    mkdir -p ${SPARK_HOME}/work-dir && \
    touch ${SPARK_HOME}/RELEASE && \
    rm /bin/sh && \
    ln -sv /bin/bash /bin/sh && \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd

# Configure Classpath environment variables
ENV INSIGHTEDGE_CLASSPATH "${GS_HOME}/insightedge/lib/*:${GS_HOME}/insightedge/lib/jdbc/*:${GS_HOME}/insightedge/lib/analytics-xtreme/*:${GS_HOME}/lib/required/*:${GS_HOME}/lib/optional/spatial/*"

ENV SPARK_CLASSPATH "${SPARK_CLASSPATH}:${INSIGHTEDGE_CLASSPATH}"
ENV SPARK_DIST_CLASSPATH "${SPARK_CLASSPATH}"

# Configure InsightEdge application's environment variables
ENV INSIGHTEDGE_SPACE_NAME insightedge-space

# Add entrypoints to directory structure
ADD docker-entrypoint.sh /opt/docker-entrypoint.sh
ADD entrypoint.sh /opt/entrypoint.sh
RUN cp /opt/gigaspaces/insightedge/spark/kubernetes/dockerfiles/spark/entrypoint.sh /opt/spark_entrypoint.sh
RUN chmod +x /opt/docker-entrypoint.sh
RUN chmod +x /opt/entrypoint.sh
RUN chmod +x /opt/spark_entrypoint.sh

# Configure Docker entrypoint
ENTRYPOINT [ "/opt/docker-entrypoint.sh" ]
CMD ["host", "run-agent", "--auto"]
