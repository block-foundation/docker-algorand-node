# syntax=docker/dockerfile:1.4


# Base Image
# =============================================================================

ARG GO_VERSION=1.17.5

FROM golang:$GO_VERSION-bullseye as build
# FROM ubuntu:22.04

# Labels
# =============================================================================


# Layer | Dependencies
# =============================================================================

RUN \
    echo 'APT::Install-Suggests "0";' >> /etc/apt/apt.conf.d/00-docker && \
    RUN echo 'APT::Install-Recommends "0";' >> /etc/apt/apt.conf.d/00-docker && \
    DEBIAN_FRONTEND=noninteractive && \
    echo 'Install System Dependencies' && \
    apt-get -q update && \
    apt-get -y upgrade && \
    apt-get install -yq --no-install-recommends \
        build-essential \
        # wget \
        # git \
        # curl \
        # g++ \
        # cmake \
        && \
    echo 'Install Python Dependencies' && \
        python3-dev \
        python3-pip \
        python3-venv \
        && \
    echo 'Install Algorand Dependencies' && \
        patch \
        libtool \
        libboost-math-dev \
        expect \
        jq \
        autoconf \
        shellcheck \
        sqlite3

# Layer | Clean System
# =============================================================================

RUN echo 'Clean apt-get' \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/* \
    && rm -rf /var/tmp/*
    




# =============================================================================
# RUNNER
# =============================================================================

FROM builder as runner
























# Basic dependencies.
ENV DEBIAN_FRONTEND noninteractive


# dependencies = patch + those listed here: https://github.com/algorand/go-algorand/blob/rel/stable/scripts/install_linux_deps.sh#L17
# Note that those get installed through configure_dev.sh anyway, but if we're rebuilding locally - for example to test out
# different branches -  it's faster to have them installed here and be cached
RUN apt-get update && apt-get install -y patch libtool libboost-math-dev expect jq autoconf shellcheck sqlite3 python3-venv

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

# Configure network
ARG ALGOD_PORT="4001"
ARG KMD_PORT="4002"
ARG TOKEN="aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
ARG TEMPLATE="DevModeNetwork.json"

COPY ["$TEMPLATE", "${BASE_DIR}/"]

ENV NETWORK_DIR="$BASE_DIR/testnetwork"
ENV NODE_DIR="${NETWORK_DIR}/Node"
ENV KMD_DATA="${NODE_DIR}/kmd-v0.5"
RUN goal network create -n sandnet -r "${NETWORK_DIR}" -t "${BASE_DIR}/${TEMPLATE}"
RUN echo $TOKEN > "${NODE_DIR}/algod.token"
RUN echo $TOKEN > "${KMD_DATA}/kmd.token"
RUN echo "{ \"Version\": 12, \"GossipFanout\": 1, \"EndpointAddress\": \"0.0.0.0:${ALGOD_PORT}\", \"DNSBootstrapID\": \"\", \"IncomingConnectionsLimit\": 0, \"Archival\":false, \"isIndexerActive\":false, \"EnableDeveloperAPI\":true}" > "${NODE_DIR}/config.json"
RUN echo "{ \"address\":\"0.0.0.0:${KMD_PORT}\",  \"allowed_origins\":[\"*\"]}" > "${KMD_DATA}/kmd_config.json"

COPY ["start.sh", "${BIN_DIR}/"]
RUN chmod +x "${BIN_DIR}/start.sh"


FROM debian:bullseye-slim

ENV PATH="/opt/algorand/bin:${PATH}" \
    ALGORAND_DATA="/opt/algorand/testnetwork/Node" \
    KMD_DATA="/opt/algorand/testnetwork/Node/kmd-v0.5"

COPY --from=build /opt/algorand /opt/algorand

# 9392 = default port for tealdbg
EXPOSE $ALGOD_PORT $KMD_PORT 9392

# Start algod
CMD ["start.sh"]











# sudo apt-get update
# sudo apt-get install -y gnupg2 curl software-properties-common
# curl -o - https://releases.algorand.com/key.pub | sudo tee /etc/apt/trusted.gpg.d/algorand.asc
# sudo add-apt-repository "deb [arch=amd64] https://releases.algorand.com/deb/ stable main"
# sudo apt-get update

# # To get both algorand and the devtools:
# sudo apt-get install -y algorand-devtools

# # Or, to only install algorand:
# sudo apt-get install -y algorand

# algod -v



# EXPOSE 8080

# RUN mkdir /algorand

# WORKDIR /algorand

# RUN apt-get update -y && apt-get install --no-install-recommends -y -q curl build-essential ca-certificates git mercurial runit

# RUN curl -L https://github.com/algorand/go-algorand-doc/raw/master/downloads/installers/linux_amd64/install_master_linux-amd64.tar.gz --output /algorand/install_master_linux-amd64.tar.gz

# RUN tar -xf install_master_linux-amd64.tar.gz

# RUN mkdir node

# RUN ./update.sh -i -c stable -p node -d node/data -n

# RUN cp node/genesisfiles/mainnet/genesis.json node/data

# ENTRYPOINT ["sh", "-c", "./update.sh -u -c stable -p node -d node/data -n && /algorand/node/algod -l 0.0.0.0:8080 -d /algorand/node/data"]


