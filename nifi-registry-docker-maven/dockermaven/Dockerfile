# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements. See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership. The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License. You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied. See the License for the
# specific language governing permissions and limitations
# under the License.
#

FROM openjdk:8-jre AS artifactbase
LABEL maintainer="Apache NiFi <dev@nifi.apache.org>"

ARG NIFI_REGISTRY_BINARY
ARG NIFI_REGISTRY_VERSION
ARG NIFI_REGISTRY_TOOLKIT_BINARY

ENV NIFI_REGISTRY_BASE_DIR /opt/nifi-registry
ENV NIFI_REGISTRY_HOME ${NIFI_REGISTRY_BASE_DIR}/nifi-registry-current
ENV NIFI_REGISTRY_TOOLKIT_HOME ${NIFI_REGISTRY_BASE_DIR}/nifi-registry-toolkit-current
ENV NIFI_REGISTRY_PID_DIR ${NIFI_REGISTRY_HOME}/run
ENV NIFI_REGISTRY_LOG_DIR ${NIFI_REGISTRY_HOME}/logs

ADD sh/ ${NIFI_REGISTRY_BASE_DIR}/scripts/
RUN chmod -R +x ${NIFI_REGISTRY_BASE_DIR}/scripts/*.sh

COPY $NIFI_REGISTRY_BINARY $NIFI_REGISTRY_BASE_DIR
RUN unzip ${NIFI_REGISTRY_BASE_DIR}/nifi-registry-${NIFI_REGISTRY_VERSION}-bin.zip -d ${NIFI_REGISTRY_BASE_DIR} \
    && rm ${NIFI_REGISTRY_BASE_DIR}/nifi-registry-${NIFI_REGISTRY_VERSION}-bin.zip \
    && mv ${NIFI_REGISTRY_BASE_DIR}/nifi-registry-${NIFI_REGISTRY_VERSION} ${NIFI_REGISTRY_HOME} \
    && ln -s ${NIFI_REGISTRY_HOME} ${NIFI_REGISTRY_BASE_DIR}/nifi-registry-${NIFI_REGISTRY_VERSION}

COPY $NIFI_REGISTRY_TOOLKIT_BINARY $NIFI_REGISTRY_BASE_DIR
RUN unzip ${NIFI_REGISTRY_BASE_DIR}/nifi-registry-toolkit-${NIFI_REGISTRY_VERSION}-bin.zip -d ${NIFI_REGISTRY_BASE_DIR} \
    && rm ${NIFI_REGISTRY_BASE_DIR}/nifi-registry-toolkit-${NIFI_REGISTRY_VERSION}-bin.zip \
    && mv ${NIFI_REGISTRY_BASE_DIR}/nifi-registry-toolkit-${NIFI_REGISTRY_VERSION} ${NIFI_REGISTRY_TOOLKIT_HOME} \
    && ln -s ${NIFI_REGISTRY_TOOLKIT_HOME} ${NIFI_REGISTRY_BASE_DIR}/nifi-registry-toolkit-${NIFI_VERSION}

# Create necessary directories
RUN mkdir -p ${NIFI_REGISTRY_HOME}/conf \
    && mkdir -p ${NIFI_REGISTRY_LOG_DIR}

FROM openjdk:8-jre
LABEL maintainer="Apache NiFi Registry <dev@nifi.apache.org>"
LABEL site="https://nifi.apache.org"

ARG UID=1000
ARG GID=1000

ENV NIFI_REGISTRY_BASE_DIR /opt/nifi-registry
ENV NIFI_REGISTRY_HOME ${NIFI_REGISTRY_BASE_DIR}/nifi-registry-current
ENV NIFI_REGISTRY_TOOLKIT_HOME ${NIFI_REGISTRY_BASE_DIR}/nifi-registry-toolkit-current
ENV NIFI_REGISTRY_PID_DIR ${NIFI_REGISTRY_HOME}/run
ENV NIFI_REGISTRY_LOG_DIR ${NIFI_REGISTRY_HOME}/logs

# Setup NiFi user and create necessary directories
RUN groupadd -g ${GID} nifi || groupmod -n nifi `getent group ${GID} | cut -d: -f1` \
    && useradd --shell /bin/bash -u ${UID} -g ${GID} -m nifi \
    && apt-get update \
    && apt-get install -y jq xmlstarlet procps

COPY --chown=nifi:nifi --from=artifactbase $NIFI_REGISTRY_BASE_DIR $NIFI_REGISTRY_BASE_DIR

VOLUME ${NIFI_REGISTRY_LOG_DIR}

USER nifi

# Web HTTP(s) ports
EXPOSE 18080 18443

WORKDIR ${NIFI_REGISTRY_HOME}

# Apply configuration and start NiFi
#
# We need to use the exec form to avoid running our command in a subshell and omitting signals,
# thus being unable to shut down gracefully:
# https://docs.docker.com/engine/reference/builder/#entrypoint
#
# Also we need to use relative path, because the exec form does not invoke a command shell,
# thus normal shell processing does not happen:
# https://docs.docker.com/engine/reference/builder/#exec-form-entrypoint-example
ENTRYPOINT ["../scripts/start.sh"]
