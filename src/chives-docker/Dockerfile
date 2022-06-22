# CHIVES BUILD STEP
FROM python:3.9 AS chives_build

ARG BRANCH=main
ARG COMMIT=""

RUN DEBIAN_FRONTEND=noninteractive apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y \
        lsb-release sudo

WORKDIR /chives-blockchain

RUN echo "cloning ${BRANCH}" && \
    git clone --branch ${BRANCH} --recurse-submodules=mozilla-ca https://github.com/HiveProject2021/chives-blockchain.git . && \
    # If COMMIT is set, check out that commit, otherwise just continue
    ( [ ! -z "$COMMIT" ] && git checkout $COMMIT ) || true && \
    echo "running build-script" && \
    /bin/sh ./install.sh

# IMAGE BUILD
FROM python:3.9-slim

EXPOSE 9755 9699 9647

ENV CHIVES_ROOT=/root/.chives/mainnet
ENV keys="generate"
ENV service="farmer"
ENV plots_dir="/plots"
ENV farmer_address=
ENV farmer_port=
ENV testnet="false"
ENV TZ="UTC"
ENV upnp="true"
ENV log_to_file="true"
ENV healthcheck="true"

# Deprecated legacy options
ENV harvester="false"
ENV farmer="false"

# Minimal list of software dependencies
#   sudo: Needed for alternative plotter install
#   tzdata: Setting the timezone
#   curl: Health-checks
RUN DEBIAN_FRONTEND=noninteractive apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y sudo tzdata curl && \
    rm -rf /var/lib/apt/lists/* && \
    ln -snf "/usr/share/zoneinfo/$TZ" /etc/localtime && echo "$TZ" > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata

COPY --from=chives_build /chives-blockchain /chives-blockchain

ENV PATH=/chives-blockchain/venv/bin:$PATH
WORKDIR /chives-blockchain

COPY docker-start.sh /usr/local/bin/
COPY docker-entrypoint.sh /usr/local/bin/
COPY docker-healthcheck.sh /usr/local/bin/

HEALTHCHECK --interval=1m --timeout=10s --start-period=20m \
  CMD /bin/bash /usr/local/bin/docker-healthcheck.sh || exit 1

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["docker-start.sh"]
