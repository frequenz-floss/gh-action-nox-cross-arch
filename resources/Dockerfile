# License: MIT
# Copyright © 2024 Frequenz Energy-as-a-Service GmbH

# This Dockerfile is used to run the tests in arm64, which is not supported by
# GitHub Actions at the moment.

ARG UBUNTU_VERSION
ARG PYTHON_VERSION

FROM docker.io/library/ubuntu:${UBUNTU_VERSION}

ENV DEBIAN_FRONTEND=noninteractive

# Install Python and curl to install pip later
RUN apt-get update -y && \
    apt-get install --no-install-recommends -y \
        software-properties-common && \
    add-apt-repository ppa:deadsnakes/ppa && \
    apt-get install --no-install-recommends -y \
        ca-certificates \
        curl \
        git \
        python${PYTHON_VERSION} \
        python${PYTHON_VERSION}-distutils && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install pip
RUN curl -sS https://bootstrap.pypa.io/get-pip.py | python${PYTHON_VERSION}

RUN update-alternatives --install \
        /usr/local/bin/python python /usr/bin/python${PYTHON_VERSION} 1 && \
    python -m pip install --upgrade --no-cache-dir pip

COPY resources/run_nox_session /usr/local/bin/run_nox_session

ENTRYPOINT ["/usr/local/bin/run_nox_session"]
