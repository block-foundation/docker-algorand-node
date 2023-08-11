# syntax=docker/dockerfile:1.4


# Base Image
# =============================================================================

ARG GO_VERSION=1.20.5
FROM golang:$GO_VERSION


# Labels
# =============================================================================

LABEL \
    maintainer="Block Foundation <info@blockfoundation.io>" \
    org.opencontainers.image.title="Algorand Node" \
    org.opencontainers.image.version="1.0" \
    org.opencontainers.image.authors="Block Foundation" \
    org.opencontainers.image.vendor="Block Foundation" \
    org.opencontainers.image.url="https://www.blockfoundation.io" \
    org.opencontainers.image.source="https://github.com/block-foundation/docker-algorand-node/" \
    org.opencontainers.image.documentation="https://github.com/block-foundation/docker-algorand-node/" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.description="Algorand Node" \
    org.opencontainers.image.ref.name="docker-algorand-node"


# Arguments
# =============================================================================

ARG CHANNEL=nightly
ARG URL=
ARG BRANCH=
ARG SHA=

# When these are set attempt to connect to a network.
ARG GENESIS_FILE=""
ARG BOOTSTRAP_URL=""

# Options for algod config
ARG ALGOD_PORT=""
ARG ALGOD_FOLLOWER_PORT=""
ARG KMD_PORT=""
ARG TOKEN=""
ARG TEMPLATE=""
ARG NETWORK_NUM_ROUNDS="30000"
ARG NODE_ARCHIVAL=""

RUN echo "Installing from source. ${URL} -- ${BRANCH}"
ENV BIN_DIR="$HOME/node"
ENV ALGORAND_DATA="/opt/data"


# Basic Dependencies
# =============================================================================

ENV HOME /opt
ENV DEBIAN_FRONTEND noninteractive

RUN \
    apt-get update && \
    apt-get install -y \
        apt-utils \
        curl \
        git \
        git-core \
        bsdmainutils \
        python3 \
        # Additional
        # dependencies = patch + those listed here: https://github.com/algorand/go-algorand/blob/rel/stable/scripts/install_linux_deps.sh#L17
        # Note that those get installed through configure_dev.sh anyway, but if we're rebuilding locally - for example to test out
        # different branches -  it's faster to have them installed here and be cached
        patch \
        libtool \
        libboost-math-dev \
        expect \
        jq \
        autoconf \
        shellcheck \
        sqlite3 \
        python3-venv



# Copy
# =============================================================================

# Copy lots of things into the container. The gitignore indicates which directories.
COPY . /tmp


# Install Algod binaries
# =============================================================================

# RUN /tmp/images/algod/install.sh \
#     -d "${BIN_DIR}" \
#     -c "${CHANNEL}" \
#     -u "${URL}" \
#     -b "${BRANCH}" \
#     -s "${SHA}"

# Install algod binaries.
ARG URL="https://github.com/algorand/go-algorand"
ARG BRANCH="rel/stable"
RUN echo "Installing from source. ${URL} -- ${BRANCH}"

RUN git clone --single-branch --branch "${BRANCH}" "${URL}" /src
WORKDIR /src
RUN ./scripts/configure_dev.sh
RUN make build

ENV BASE_DIR="/opt/algorand"
ENV BIN_DIR="$BASE_DIR/bin"
ENV PATH="$BIN_DIR:${PATH}"

RUN ./scripts/dev_install.sh -p ${BIN_DIR}


# Configure Network
# =============================================================================

RUN sed -i \
  -e "s/NETWORK_NUM_ROUNDS/$NETWORK_NUM_ROUNDS/" \
  -e "s/ALGOD_FOLLOWER_PORT/$ALGOD_FOLLOWER_PORT/" \
  "//tmp/${TEMPLATE}" && \
 /tmp/images/algod/setup.py \
 --bin-dir "$BIN_DIR" \
 --data-dir "/opt/data" \
 --start-script "/opt/start_algod.sh" \
 --network-dir "/opt/testnetwork" \
 --network-template "//tmp/${TEMPLATE}" \
 --network-token "${TOKEN}" \
 --algod-port "${ALGOD_PORT}" \
 --kmd-port "${KMD_PORT}" \
 --bootstrap-url "${BOOTSTRAP_URL}" \
 --genesis-file "/tmp/${GENESIS_FILE}" \
 --archival "${NODE_ARCHIVAL}"


# User Environment
# =============================================================================

ENV PATH="$BIN_DIR:${PATH}"
WORKDIR /opt/data


# Start algod
# =============================================================================

CMD ["/opt/start_algod.sh"]