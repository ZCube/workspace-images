# Copyright (c) 2020 Gitpod GmbH. All rights reserved.
# Licensed under the GNU Affero General Public License (AGPL).
# See License-AGPL.txt in the project root for license information.

# if you want to run this image in a workspace, follow these steps:
# gcloud auth configure-docker europe-docker.pkg.dev
# gcloud auth login --no-launch-browser
FROM ghcr.io/zcube/workspace-images/full:arm64

ENV TRIGGER_REBUILD 16

USER root

### cloud_sql_proxy ###
ARG CLOUD_SQL_PROXY=/usr/local/bin/cloud_sql_proxy
RUN curl -fsSL https://dl.google.com/cloudsql/cloud_sql_proxy.linux.$(dpkg --print-architecture) > $CLOUD_SQL_PROXY \
    && chmod +x $CLOUD_SQL_PROXY

### Helm3 ###
RUN mkdir -p /tmp/helm/ \
    && curl -fsSL https://get.helm.sh/helm-v3.7.1-linux-$(dpkg --print-architecture).tar.gz | tar -xzvC /tmp/helm/ --strip-components=1 \
    && cp /tmp/helm/helm /usr/local/bin/helm \
    && ln -s /usr/local/bin/helm /usr/local/bin/helm3 \
    && rm -rf /tmp/helm/ \
    && helm completion bash > /usr/share/bash-completion/completions/helm

### kubernetes ###
RUN mkdir -p /usr/local/kubernetes/ && \
    curl -fsSL https://github.com/kubernetes/kubernetes/releases/download/v1.22.4/kubernetes.tar.gz \
    | tar -xzvC /usr/local/kubernetes/ --strip-components=1 \
    && KUBERNETES_SKIP_CONFIRM=true /usr/local/kubernetes/cluster/get-kube-binaries.sh \
    && chown gitpod:gitpod -R /usr/local/kubernetes

ENV PATH=$PATH:/usr/local/kubernetes/cluster/:/usr/local/kubernetes/client/bin/

RUN cat /var/lib/dpkg/status
### kubectl ###
RUN curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - \
    # really 'xenial'
    && add-apt-repository -yu "deb https://apt.kubernetes.io/ kubernetes-xenial main" \
    && install-packages kubectl=1.22.4-00 \
    && kubectl completion bash > /usr/share/bash-completion/completions/kubectl

RUN curl -fsSL -o /usr/bin/kubectx https://raw.githubusercontent.com/ahmetb/kubectx/master/kubectx && chmod +x /usr/bin/kubectx \
    && curl -fsSL -o /usr/bin/kubens  https://raw.githubusercontent.com/ahmetb/kubectx/master/kubens  && chmod +x /usr/bin/kubens

RUN curl -fsSL https://github.com/kubernetes-sigs/kubebuilder/releases/download/v2.3.2/kubebuilder_2.3.2_linux_$(dpkg --print-architecture).tar.gz | tar -xz -C /tmp/ \
    && sudo mkdir -p /usr/local/kubebuilder \
    && sudo mv /tmp/kubebuilder_2.3.2_linux_$(dpkg --print-architecture)/* /usr/local/kubebuilder \
    && rm -rf /tmp/*

### MySQL client ###
RUN install-packages mysql-client

# golangci-lint
RUN cd /usr/local && curl -fsSL https://install.goreleaser.com/github.com/golangci/golangci-lint.sh | sh -s v1.42.0

# gokart
#RUN cd /usr/bin && curl -L https://github.com/praetorian-inc/gokart/releases/download/v0.3.0/gokart_0.3.0_linux_$(uname -m).tar.gz | tar xzv gokart
RUN go install github.com/praetorian-inc/gokart@latest \
    && sudo mv $GOPATH/bin/gokart /usr/bin/gokart \
    && sudo chown root:root /usr/bin/gokart \
    && sudo rm -rf $GOPATH $GOPATH/src $GOPATH/pkg /home/gitpod/.cache/go /home/gitpod/.cache/go-build \
    && sudo chown gitpod:gitpod /workspace

# leeway
ENV LEEWAY_MAX_PROVENANCE_BUNDLE_SIZE=8388608
RUN arch="$(uname -m)"; \
	url=; \
	case "$arch" in \
		'x86_64') \
			export ARCH1='x86_64' \
			;; \
		'aarch64') \
			export ARCH1='arm64' \
			;; \
		*) echo >&2 "error: unsupported architecture '$arch'"; exit 1 ;; \
	esac; \
    cd /usr/bin && curl -fsSL https://github.com/gitpod-io/leeway/releases/download/v0.2.16/leeway_0.2.16_Linux_${ARCH1}.tar.gz | tar xz

# dazzle
RUN arch="$(uname -m)"; \
	url=; \
	case "$arch" in \
		'x86_64') \
			export ARCH1='x86_64' \
			;; \
		'aarch64') \
			export ARCH1='arm64' \
			;; \
		*) echo >&2 "error: unsupported architecture '$arch'"; exit 1 ;; \
	esac; \
    cd /usr/bin && curl -fsSL https://github.com/gitpod-io/dazzle/releases/download/v0.1.6/dazzle_0.1.6_Linux_${ARCH1}.tar.gz| tar xz

# werft CLI
ENV WERFT_K8S_NAMESPACE=werft
ENV WERFT_DIAL_MODE=kubernetes
RUN git clone -b v0.1.5 https://github.com/csweichel/werft \
    && cd werft \
    && go build --tags client -o werft client.go \
    && cd ..\
    && sudo mv werft/werft /usr/bin/werft \
    && sudo chown root:root /usr/bin/werft \
    && sudo rm -rf $GOPATH $GOPATH/src $GOPATH/pkg /home/gitpod/.cache/go /home/gitpod/.cache/go-build werft \
    && sudo chown gitpod:gitpod /workspace
#RUN cd /usr/bin && curl -fsSL https://github.com/csweichel/werft/releases/download/v0.1.4/werft-client-linux-$(dpkg --print-architecture).tar.gz | tar xz && mv werft-client-linux-$(dpkg --print-architecture) werft

# yq - jq for YAML files
# Note: we rely on version 3.x.x in various places, 4.x breaks this!
RUN cd /usr/bin && curl -fsSL https://github.com/mikefarah/yq/releases/download/3.4.1/yq_linux_$(dpkg --print-architecture) > yq && chmod +x yq

# release helper
RUN cd /usr/bin && curl -fsSL https://github.com/c4milo/github-release/releases/download/v1.1.0/github-release_v1.1.0_linux_$(dpkg --print-architecture).tar.gz | tar xz

### Protobuf
RUN arch="$(uname -m)"; \
	url=; \
	case "$arch" in \
		'x86_64') \
			export ARCH2='x86_64' \
			;; \
		'aarch64') \
			export ARCH2='aarch_64' \
			;; \
		*) echo >&2 "error: unsupported architecture '$arch'"; exit 1 ;; \
	esac; \
    set -ex \
    && tmpdir=$(mktemp -d) \
    && curl -fsSL -o $tmpdir/protoc.zip https://github.com/protocolbuffers/protobuf/releases/download/v3.19.1/protoc-3.19.1-linux-${ARCH2}.zip \
    && mkdir -p /usr/lib/protoc && cd /usr/lib/protoc && unzip $tmpdir/protoc.zip \
    && chmod -R o+r+x /usr/lib/protoc/include \
    && chmod -R +x /usr/lib/protoc/bin \
    && ln -s /usr/lib/protoc/bin/* /usr/bin \
    && rm -rf $tmpdir

# ### Telepresence ###
# RUN curl -fsSL https://packagecloud.io/datawireio/telepresence/gpgkey | apt-key add - \
#     # 'cosmic' not supported
#     && add-apt-repository -yu "deb https://packagecloud.io/datawireio/telepresence/ubuntu/ bionic main" \
#     # 0.95 (current at the time of this commit) is broken
RUN install-packages \
    iproute2 \
    iptables \
    net-tools \
    socat
#     telepresence=0.109

### Toxiproxy CLI
RUN curl -fsSL -o /usr/bin/toxiproxy https://github.com/Shopify/toxiproxy/releases/download/v2.3.0/toxiproxy-cli-linux-$(dpkg --print-architecture) \
    && chmod +x /usr/bin/toxiproxy

### libseccomp > 2.5.2
RUN install-packages gperf \
    && cd $(mktemp -d) \
    && curl -fsSL https://github.com/seccomp/libseccomp/releases/download/v2.5.2/libseccomp-2.5.2.tar.gz | tar xz \
    && cd libseccomp-2.5.2 && ./configure && make && make install

### Cypress deps
RUN install-packages \
    libgtk2.0-0 \
    libgtk-3-0 \
    libgbm-dev \
    libnotify-dev \
    libgconf-2-4 \
    libnss3 \
    libxss1 \
    libasound2 \
    libxtst6 \
    xauth

USER gitpod

# Fix node version we develop against
ARG GITPOD_NODE_VERSION=16.13.1
RUN bash -c ". .nvm/nvm.sh \
    && nvm install $GITPOD_NODE_VERSION \
    && npm install -g typescript yarn"
ENV PATH=/home/gitpod/.nvm/versions/node/v${GITPOD_NODE_VERSION}/bin:$PATH

# Go
ENV GOFLAGS="-mod=readonly"

## Register leeway autocompletion in bashrc
RUN bash -c "echo . \<\(leeway bash-completion\) >> ~/.bashrc"

### Google Cloud ###
# not installed via repository as then 'docker-credential-gcr' is not available
ARG GCS_DIR=/opt/google-cloud-sdk
ENV PATH=$GCS_DIR/bin:$PATH
RUN arch="$(uname -m)"; \
	url=; \
	case "$arch" in \
		'x86_64') \
			export ARCH3='x86_64' \
			;; \
		'aarch64') \
			export ARCH3='arm' \
			;; \
		*) echo >&2 "error: unsupported architecture '$arch'"; exit 1 ;; \
	esac; \
    sudo chown gitpod: /opt \
    && mkdir $GCS_DIR \
    && curl -fsSL https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-354.0.0-linux-${ARCH3}.tar.gz \
    | tar -xzvC /opt \
    && /opt/google-cloud-sdk/install.sh --quiet --usage-reporting=false --bash-completion=true \
    --additional-components docker-credential-gcr alpha beta \
    # needed for access to our private registries
    && docker-credential-gcr configure-docker

# Install tools for gsutil
RUN sudo install-packages \
    gcc \
    python-dev \
    python-setuptools

RUN sudo python3 -m pip uninstall crcmod; sudo python3 -m pip install --no-cache-dir -U crcmod

### gitpod-core specific gcloud/kubectl config
# Copy GCloud default config that points to gitpod-dev
ARG GCLOUD_CONFIG_DIR=/home/gitpod/.config/gcloud
COPY --chown=gitpod gcloud-default-config $GCLOUD_CONFIG_DIR/configurations/config_default

# Set kubeconfig file for dev cluster, using GCloud Application Default Credentials (ADC) as auth provider
ARG KUBE_CONFIG_PATH=/home/gitpod/.kube/config
COPY --chown=gitpod kubeconfig.yaml $KUBE_CONFIG_PATH

# Set Application Default Credentials (ADC) based on user-provided env var
RUN echo ". /workspace/gitpod/scripts/setup-google-adc.sh" >> ~/.bashrc

ENV DB_HOST=localhost

ENV LEEWAY_WORKSPACE_ROOT=/workspace/gitpod
ENV LEEWAY_REMOTE_CACHE_BUCKET=gitpod-core-leeway-cache-branch

### AWS Cli ###
RUN sudo python3 -m pip install --no-cache-dir awscli

# Install aws-iam-authenticator
RUN sudo curl -fsSL -o aws-iam-authenticator "https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/$(dpkg --print-architecture)/aws-iam-authenticator" \
    && sudo chmod +x ./aws-iam-authenticator \
    && sudo mkdir -p $HOME/.aws-iam \
    && sudo mv ./aws-iam-authenticator $HOME/.aws-iam/aws-iam-authenticator \
    && sudo chown -R gitpod:gitpod $HOME/.aws-iam

# Install Terraform
ARG RELEASE_URL="https://releases.hashicorp.com/terraform/0.15.4/terraform_0.15.4_linux_$(dpkg --print-architecture).zip"
RUN mkdir -p ~/.terraform \
    && cd ~/.terraform \
    && curl -fsSL -o terraform_linux_$(dpkg --print-architecture).zip ${RELEASE_URL} \
    && unzip *.zip \
    && rm -f *.zip \
    && printf "terraform -install-autocomplete 2> /dev/null\n" >>~/.bashrc

# Install GraphViz to help debug terraform scripts
RUN sudo install-packages graphviz

ENV PATH=$PATH:$HOME/.aws-iam:$HOME/.terraform

# Install codecov uploader
# https://about.codecov.io/blog/introducing-codecovs-new-uploader
RUN sudo curl -fsSL https://uploader.codecov.io/latest/codecov-linux -o /usr/local/bin/codecov \
    && sudo chmod +x /usr/local/bin/codecov

# Install pre-commit https://pre-commit.com/#install
RUN sudo install-packages shellcheck \
    && sudo python3 -m pip install pre-commit

# Install observability-related binaries
ARG PROM_VERSION="2.30.0"
RUN curl -LO https://github.com/prometheus/prometheus/releases/download/v${PROM_VERSION}/prometheus-${PROM_VERSION}.linux-$(dpkg --print-architecture).tar.gz && \
    tar -xzvf prometheus-${PROM_VERSION}.linux-$(dpkg --print-architecture).tar.gz && \
    sudo mv prometheus-${PROM_VERSION}.linux-$(dpkg --print-architecture)/promtool /usr/local/bin/promtool && \
    rm -rf prometheus-${PROM_VERSION}.linux-$(dpkg --print-architecture)/

ARG JSONNET_BUNDLER_VERSION="0.4.0"
RUN curl -L -o jb https://github.com/jsonnet-bundler/jsonnet-bundler/releases/download/v${JSONNET_BUNDLER_VERSION}/jb-linux-$(dpkg --print-architecture) && \
    chmod +x jb && sudo mv jb /usr/local/bin

ARG JSONNET_VERSION="0.18.0"
RUN arch="$(uname -m)"; \
	url=; \
	case "$arch" in \
		'x86_64') \
			export ARCH1='x86_64' \
			;; \
		'aarch64') \
			export ARCH1='arm64' \
			;; \
		*) echo >&2 "error: unsupported architecture '$arch'"; exit 1 ;; \
	esac; \
    curl -LO https://github.com/google/go-jsonnet/releases/download/v${JSONNET_VERSION}/go-jsonnet_${JSONNET_VERSION}_Linux_${ARCH1}.tar.gz && \
    tar -xzvf go-jsonnet_${JSONNET_VERSION}_Linux_${ARCH1}.tar.gz && \
    sudo mv jsonnet /usr/local/bin/jsonnet && \
    sudo mv jsonnetfmt /usr/local/bin/jsonnetfmt

ARG GOJSONTOYAML_VERSION="0.1.0"
RUN curl -LO https://github.com/brancz/gojsontoyaml/releases/download/v${GOJSONTOYAML_VERSION}/gojsontoyaml_${GOJSONTOYAML_VERSION}_linux_$(dpkg --print-architecture).tar.gz && \
    tar -xzvf gojsontoyaml_${GOJSONTOYAML_VERSION}_linux_$(dpkg --print-architecture).tar.gz && \
    sudo mv gojsontoyaml /usr/local/bin/gojsontoyaml

RUN curl -LO https://dl.min.io/client/mc/release/linux-$(dpkg --print-architecture)/mc && \
    chmod +x mc && sudo mv mc /usr/local/bin

# Copy our own tools
#COPY dev-kubecdl--app/kubecdl dev-gpctl--app/gpctl /usr/bin/
