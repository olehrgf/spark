#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
ARG java_image_tag=17-al2023-headless

FROM public.ecr.aws/amazoncorretto/amazoncorretto:${java_image_tag}

ARG spark_uid=185

ARG USER_UID="185"
ARG USER_GID="185"
ARG USER_NAME="spark"
ARG TARGETARCH

# Before building the docker image, first build and make a Spark distribution following
# the instructions in https://spark.apache.org/docs/latest/building-spark.html.
# If this docker file is being used in the context of building your images from a Spark
# distribution, the docker build command should be invoked from the top level directory
# of the Spark distribution. E.g.:
# docker build -t spark:latest -f kubernetes/dockerfiles/spark/Dockerfile .

RUN set -ex && \
    dnf -y update && \
    rm /bin/sh && ln -s /bin/bash /bin/sh && ln -s /lib /lib64 && \
    dnf -y install dnf-utils openssl pam sssd realmd krb5-workstation samba-common-tools nss procps net-tools aws-cli && \
    dnf -y install glibc-locale-source glibc-langpack-en && \
    localedef -i en_US -f UTF-8 en_US.UTF-8 && \
    groupadd -g $USER_GID $USER_NAME && \
    useradd -m -g $USER_GID -u $USER_UID $USER_NAME && \
    chown -R $USER_NAME:$USER_GID /home/$USER_NAME && \
    mkdir -p /opt/spark && \
    mkdir -p /opt/spark/work-dir && \
    mkdir -p /etc/metrics/conf && \
    touch /opt/spark/RELEASE && \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd && \
    rm -rf /var/cache/apt/* && rm -rf /var/lib/apt/lists/* && \
    dnf clean all && \
    rm -rf /var/cache/dnf \

RUN curl -L -o tini "https://github.com/krallin/tini/releases/download/v0.19.0/tini-${TARGETARCH}" && \
    chmod +x ./tini && mv tini /usr/bin/tini && \
    curl -L -o tini-static "https://github.com/krallin/tini/releases/download/v0.19.0/tini-static-${TARGETARCH}" && \
    chmod +x ./tini-static && mv tini-static /usr/bin/tini-static

COPY jars /opt/spark/jars
# Copy RELEASE file if exists
COPY RELEAS[E] /opt/spark/RELEASE
COPY bin /opt/spark/bin
COPY sbin /opt/spark/sbin
COPY kubernetes/dockerfiles/spark/entrypoint.sh /opt/
COPY kubernetes/dockerfiles/spark/decom.sh /opt/
#COPY examples /opt/spark/examples
COPY kubernetes/tests /opt/spark/tests
#COPY data /opt/spark/data
COPY conf/metrics.properties /etc/metrics/conf
COPY conf/prometheus.yaml /etc/metrics/conf
#COPY data /opt/spark/data
COPY kubernetes/dockerfiles/spark/conf/metrics.properties /etc/metrics/conf
COPY kubernetes/dockerfiles/spark/conf/prometheus.yaml /etc/metrics/conf

ENV SPARK_HOME /opt/spark

RUN chmod g+w /opt/spark/work-dir
RUN chmod a+x /opt/decom.sh
RUN echo "$USER_UID:x:$USER_UID:$USER_GID:anonymous uid:$SPARK_HOME:/bin/false" >> /etc/passwd

WORKDIR /opt/spark/work-dir

ENTRYPOINT [ "/opt/entrypoint.sh" ]

# Specify the User that the actual main process will run as
USER ${USER_UID}
