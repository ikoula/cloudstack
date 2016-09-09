# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
# 
#   http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
#
# CloudStack-simulator build

FROM phusion/baseimage:latest

MAINTAINER "IKOULA's CloudStack Team" <cloudstack@ikoula.com>
LABEL Vendor="Apache.org" License="ApacheV2" Version="4.6.0"

RUN apt-get -y update && apt-get install -y \
    build-essential \
    genisoimage \
    git \
    libssl-dev \
    libffi-dev \
    maven \
    openjdk-8-jdk \
    python-dev \
    python-setuptools \
    python-pip \
    python-mysql.connector \
    supervisor \
    wget

RUN echo 'mysql-server mysql-server/root_password password root' |  debconf-set-selections; \
    echo 'mysql-server mysql-server/root_password_again password root' |  debconf-set-selections;

RUN apt-get install -qqy mysql-server && \
    apt-get clean all

RUN (/usr/bin/mysqld_safe &); sleep 5; mysqladmin -u root -proot password ''

RUN wget -nv https://raw.githubusercontent.com/ikoula/cloudstack/master/tools/docker/supervisord.conf -O /etc/supervisor/conf.d/supervisord.conf

WORKDIR /root

RUN git clone https://github.com/ikoula/cloudstack.git

WORKDIR /root/cloudstack

RUN git checkout tags/4.9.0

RUN mvn -Pdeveloper,systemvm -Dsimulator -DskipTests clean install

RUN wget -nv http://download.cloud.com.s3.amazonaws.com/tools/vhd-util -O /root/cloudstack/scripts/vm/hypervisor/xenserver/vhd-util

ENV PKG_URL=http://jenkins.buildacloud.org/job/cloudstack-marvin-master/lastSuccessfulBuild/artifact/tools/marvin/dist/Marvin-4.9.0-SNAPSHOT.tar.gz

RUN (/usr/bin/mysqld_safe &); \
    sleep 3; \
    mvn -Pdeveloper,systemvm -pl developer -Ddeploydb; \
    mvn -Pdeveloper,systemvm -pl developer -Ddeploydb-simulator; \
    pip install $PKG_URL

EXPOSE 8080 8096

CMD ["/usr/bin/supervisord"]
