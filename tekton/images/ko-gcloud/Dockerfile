# Copyright 2019-2025 The Tekton Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

ARG GO_VERSION=1.24.1
FROM ghcr.io/ko-build/ko:v0.18.0@sha256:78fa20b3bf2f56126f710f46c14efe7d71610f349ea289595050a5c9d9c7e426 AS ko
FROM golang:1.24.3-alpine@sha256:b4f875e650466fa0fe62c6fd3f02517a392123eea85f1d7e69d85f780e4db1c1 AS build
LABEL description="Build container"

RUN apk update && apk add --no-cache alpine-sdk ca-certificates
RUN update-ca-certificates

ARG KUBECTL_VERSION=1.28.11
RUN wget -O/usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/v$KUBECTL_VERSION/bin/linux/amd64/kubectl; chmod +x /usr/local/bin/kubectl

# Install Kustomize
ENV GOBIN=/usr/local/go/bin
ENV GO111MODULE on
RUN go install sigs.k8s.io/kustomize/kustomize/v5@v5.4.2

# Install yq for YAML parsing
RUN apk add --no-cache yq

FROM google/cloud-sdk:524.0.0-alpine@sha256:cad12907540b1a43c9279503796723817e62da1f8fd3b8723755effb9d55e1e1
ARG GO_VERSION
LABEL maintainer "Tekton Authors <tekton-dev@googlegroups.com>"
LABEL org.opencontainers.image.source=https://github.com/tektoncd/plumbing
LABEL org.opencontainers.image.description="Build Container for Ko"
LABEL org.opencontainers.image.licenses=Apache-2.0

# Install golang
RUN curl https://dl.google.com/go/go${GO_VERSION}.linux-amd64.tar.gz > go${GO_VERSION}.tar.gz
RUN tar -C /usr/local -xzf go${GO_VERSION}.tar.gz
ENV PATH="${PATH}:/usr/local/go/bin"
ENV GOROOT /usr/local/go

# Get static binaries from the build image
COPY --from=ko /ko-app/ko /usr/local/bin/ko
COPY --from=build /usr/local/bin/kubectl /usr/local/bin/kubectl
COPY --from=build /usr/local/go/bin/kustomize /usr/local/go/bin
COPY --from=build /usr/bin/yq /usr/local/bin/yq
