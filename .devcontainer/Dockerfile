#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

FROM mcr.microsoft.com/vscode/devcontainers/javascript-node:14

# The javascript-node image includes a non-root node user with sudo access. Use 
# the "remoteUser" property in devcontainer.json to use it. On Linux, the container 
# user's GID/UIDs will be updated to match your local UID/GID when using the image
# or dockerFile property. Update USER_UID/USER_GID below if you are using the
# dockerComposeFile property or want the image itself to start with different ID
# values. See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=node
ARG USER_UID=1000
ARG USER_GID=$USER_UID
# ARG EXTRA_NODE_VERSION=14
ENV KUBECONFIG="/usr/local/share/kube-devcontainer/kubeconfig.yml"
ENV XSCLI="/workspaces//home/sap-tech-bytes/XS_CLIENT00P_134-70001320"
ENV PATH=$PATH:${XSCLI}/bin
ENV npm config set registry https://registry.npmjs.org/

# # # Prepare for apt-based install of Cloud Foundry CLI by adding Cloud Foundry Foundation public key & package repository
# # # (see https://docs.cloudfoundry.org/cf-cli/install-go-cli.html#pkg-linux).
RUN wget -q -O - https://packages.cloudfoundry.org/debian/cli.cloudfoundry.org.key | sudo apt-key add - ; \
    echo "deb https://packages.cloudfoundry.org/debian stable main" | sudo tee /etc/apt/sources.list.d/cloudfoundry-cli.list

# Install an additional version of node using nvm
# RUN su node -c "source /usr/local/share/nvm/nvm.sh && nvm install ${EXTRA_NODE_VERSION}"

# # Alter node user as needed, install tslint, typescript. eslint is installed by javascript image
RUN if [ "$USER_GID" != "1000" ] || [ "$USER_UID" != "1000" ]; then \
    groupmod --gid $USER_GID $USERNAME \
    && usermod --uid $USER_UID --gid $USER_GID $USERNAME \
    && chown -R $USER_UID:$USER_GID /home/$USERNAME; \
    fi \
    #
    # Install tslint, typescript. eslint is installed by javascript image
    && sudo -u ${USERNAME} npm install -g typescript \
    # && sudo -u ${USERNAME} npm set @sap:registry=https://npm.sap.com -g\
    # Install global node modules for SAP CAP and frontend development.
    && npm config set registry https://registry.npmjs.org/ \
    && sudo -u ${USERNAME} npm i @sap/cds-dk \
    -g mta \
    -g yo \
    -g @sap/generator-base-mta-module \
    -g @sap/generator-fiori \    
    -g yo generator-easy-ui5 \
    @sapui5/generator-sapui5-templates \
    @sap/generator-add-hdb-module \
    @sap/generator-cap-project \
    @sap/generator-fiori \
    @sap/generator-hdb-project \
    hana-cli 

# # # Install kubectl (K8s)
RUN curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl \
    && chmod +x ./kubectl \
    && mv ./kubectl /usr/local/bin

# create directory for mounting .kube
RUN cd /usr/local/share && sudo -s mkdir kube-devcontainer \
    && sudo -s chmod 700 ./kube-devcontainer/ \
    && sudo -s chown --recursive ${USERNAME}:${USERNAME} ./kube-devcontainer/ 

# # # Install extra tools for CAP development & deployment.
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    && apt-get -y install sqlite cf-cli

# Install minikube
RUN curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
    && chmod +x minikube \
    && sudo -s cp minikube /usr/local/bin && rm minikube

# Install Helm
RUN  wget -q https://get.helm.sh/helm-v3.6.3-linux-amd64.tar.gz -O - | tar -xzO linux-amd64/helm > /usr/local/bin/helm \
    && chmod +x /usr/local/bin/helm \
    && chmod g+rwx /root \
    # && mkdir /config \
    # && chmod g+rwx /config \
    && helm repo add "stable" "https://charts.helm.sh/stable" --force-update

# Install docker
RUN apt-get -y update \
    && apt-get -y install curl \
    && apt-get -y install apt-transport-https \
    && apt-get -y install ca-certificates \
    && apt-get -y install gnupg \
    && apt-get -y install lsb-release \
    && apt-get -y install software-properties-common \
    && rm -rf /var/lib/apt/lists/*

## Add User node to the docker group
RUN sudo -s groupadd docker \
    && usermod -aG docker ${USERNAME} \
    && newgrp docker

## Install docker
RUN sudo -u ${USERNAME} curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add - \
    && add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" \
    && apt-get -y update \
    && apt-get -y install docker.io

# XS CLI for running against HANA XSA
RUN apt-get update \
    && apt-get install --yes --no-install-recommends procps unzip \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir --parents ${XSCLI} \
    && chown --recursive ${USERNAME}:${USERNAME} ${XSCLI}