#ARG ALPINE_VERSION=3.15
ARG NODE_VERSION=14.21.2
ARG BUILDPLATFORM=linux/amd64
FROM --platform=$BUILDPLATFORM node:${NODE_VERSION} AS node-builder

COPY . /src

WORKDIR /src/app

RUN npm install
RUN npm run build

# NOGIT: FIXME: the musl wheel for gevent isn't built on arm64
FROM ${DOCKER_REGISTRY:+$DOCKER_REGISTRY/}python:3.7-slim-bullseye AS postgres-operator-ui

LABEL maintainer="Team ACID @ Zalando <team-acid@zalando.de>"

EXPOSE 8081

RUN \
  python3 -m ensurepip && \
  pip3 install --upgrade \
    gevent \
    jq \
    pip \
    setuptools \
    && \
  rm -rf \
    /root/.cache \
    /tmp/*

COPY requirements.txt /
COPY start_server.sh /

RUN pip3 install -r /requirements.txt

COPY --from=node-builder /src/operator_ui /operator_ui

ARG VERSION=dev
RUN sed -i "s/__version__ = .*/__version__ = '${VERSION}'/" /operator_ui/__init__.py

WORKDIR /
CMD ["/usr/bin/python3", "-m", "operator_ui"]

