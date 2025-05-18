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

FROM ubuntu:20.04
LABEL maintainer="Michele Sciabarra michele@nuvolaris.io, Francesco Timperi Tiberi francesco.timperi@gmail.com"
ARG TARGETPLATFORM

# configure dpkg && timezone
ENV TZ=Europe/London
RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections ;\
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime ;\
    echo $TZ > /etc/timezone
# add docker and java (amazon corretto) repos
RUN apt-get update && apt-get -y upgrade &&\
    apt-get -y install \
    software-properties-common apt-utils \
    lsb-release ca-certificates \
    curl wget gpg sudo \
    socat telnet inetutils-ping \
    unzip vim telnet less sudo git \
    python3 python3-pip gcc g++ make \
    silversearcher-ag zip lsof
# install java (amazon corretto) and libs
RUN pip3 install --upgrade pip setuptools six 
# python2 stuff
RUN ln -sf /usr/bin/python3 /usr/bin/python ;\
    curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py ;\
    python get-pip.py ;\
    python -m pip install ansible==2.8.18 pydocumentdb humanize requests
# docker
RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu bionic stable" > /etc/apt/sources.list.d/docker.list ;\
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor > /usr/share/keyrings/docker-archive-keyring.gpg ;\
    apt-get update && apt-get -y install docker-ce-cli 
# protobuf setup
RUN apt-get -y install protobuf-compiler

# install graalvm
ENV GRAALVM_HOME=/opt/graalvm
ENV PATH=$GRAALVM_HOME/bin:$PATH
ARG GRAALVM_VERSION=22.3.3  https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-22.3.3/graalvm-ce-java11-linux-amd64-22.3.3.tar.gz

RUN echo "TARGETPLATFORM is $TARGETPLATFORM"
RUN if [ "$TARGETPLATFORM" = "linux/arm64" ]; then \
      wget https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-$GRAALVM_VERSION/graalvm-ce-java11-linux-aarch64-$GRAALVM_VERSION.tar.gz -O /tmp/graalvm.tar.gz; \
    elif [ "$TARGETPLATFORM" = "linux/amd64" ]; then \
      wget https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-$GRAALVM_VERSION/graalvm-ce-java11-linux-amd64-$GRAALVM_VERSION.tar.gz -O /tmp/graalvm.tar.gz; \
    else \
        echo "Unsupported platform: $TARGETPLATFORM"; \
        exit 1; \      
    fi && \
    # Extract GraalVM
    mkdir -p $GRAALVM_HOME && \
    tar -xvzf /tmp/graalvm.tar.gz --strip-components=1 -C $GRAALVM_HOME && \
    rm /tmp/graalvm.tar.gz && \
    # Install Native Image
    gu install native-image && \
    # Clean up
    apt-get clean

# Set GraalVM as the default Java
RUN update-alternatives --install /usr/bin/java java $GRAALVM_HOME/bin/java 1
RUN update-alternatives --install /usr/bin/javac javac $GRAALVM_HOME/bin/javac 1

# Verify GraalVM and Native Image installation
RUN java -version
RUN native-image --version
    
RUN useradd -m openserverless -s /bin/bash ;\
    echo "openserverless ALL=(ALL:ALL) NOPASSWD: ALL" >>/etc/sudoers
# node
RUN curl -sL https://deb.nodesource.com/setup_22.x | bash ;\
    apt-get install -y nodejs ;\
    curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | tee /usr/share/keyrings/yarnkey.gpg >/dev/null ;\
    echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | tee /etc/apt/sources.list.d/yarn.list ;\
    apt-get update && sudo apt-get install yarn
# task
RUN sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b /usr/local/bin
# act github actions verifier
RUN curl --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash

ENV HOME=/home/openserverless
ENV ANSIBLE_CMD="ansible-playbook -i environments/local -e docker_image_prefix=testing"
ENV GRADLE_PROJS_SKIP=""
WORKDIR /home/openserverless
ENTRYPOINT [ "/usr/bin/socat","UNIX-LISTEN:/var/run/docker.sock,fork,mode=660,user=openserverless","UNIX-CONNECT:/var/run/docker-host.sock" ]    