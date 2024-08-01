FROM debian:stable-slim

LABEL maintainer="https://github.com/factoriotools/factorio-docker"

ARG USER=factorio
ARG GROUP=factorio
ARG PUID=845
ARG PGID=845
ARG BOX64_VERSION=v0.2.4

ARG PRESET
ARG VERSION
ARG SHA256

LABEL factorio.version=${VERSION}

ARG CURL_RETRIES=8

ENV PORT=34197 \
    RCON_PORT=27015 \
    VERSION=${VERSION} \
    SHA256=${SHA256} \
    SAVES=/factorio/saves \
    PRESET="$PRESET" \
    CONFIG=/factorio/config \
    MODS=/factorio/mods \
    SCENARIOS=/factorio/scenarios \
    SCRIPTOUTPUT=/factorio/script-output \
    PUID="$PUID" \
    PGID="$PGID"

SHELL ["/bin/bash", "-eo", "pipefail", "-c"]
RUN set -ox pipefail \
    && if [[ "${VERSION}" == "" ]]; then \
        echo "build-arg VERSION is required" \
        && exit 1; \
    fi \
    && if [[ "${SHA256}" == "" ]]; then \
        echo "build-arg SHA256 is required" \
        && exit 1; \
    fi \
    && archive="/tmp/factorio_headless_x64_$VERSION.tar.xz" \
    && mkdir -p /opt /factorio \
    && apt-get -q update \
    && DEBIAN_FRONTEND=noninteractive apt-get -qy install ca-certificates curl jq pwgen xz-utils procps gettext-base --no-install-recommends \
    && curl -sSL "https://www.factorio.com/get-download/$VERSION/headless/linux64" -o "$archive" --retry $CURL_RETRIES\
    && echo "$SHA256  $archive" | sha256sum -c \
    || (sha256sum "$archive" && file "$archive" && exit 1) \
    && tar xf "$archive" --directory /opt \
    && chmod ugo=rwx /opt/factorio \
    && rm "$archive" \
    && if [[ "$(uname -m)" == "aarch64" ]]; then \
        echo "installing ARM compatability layer" \
        && DEBIAN_FRONTEND=noninteractive apt-get -qy install unzip --no-install-recommends \ 
        && curl -LO https://github.com/ptitSeb/box64/releases/download/${BOX64_VERSION}/box64-GENERIC_ARM-RelWithDebInfo.zip \
        && unzip box64-GENERIC_ARM-RelWithDebInfo.zip -d /bin \
        && rm -f box64-GENERIC_ARM-RelWithDebInfo.zip \
        && chmod +x /bin/box64; \
    fi \
    && rm -rf /var/lib/apt/lists/* \
    && ln -s "$SCENARIOS" /opt/factorio/scenarios \
    && ln -s "$SAVES" /opt/factorio/saves \
    && mkdir -p /opt/factorio/config/ \
    && addgroup --system --gid "$PGID" "$GROUP" \
    && adduser --system --uid "$PUID" --gid "$PGID" --no-create-home --disabled-password --shell /bin/sh "$USER" \
    && chown -R "$USER":"$GROUP" /opt/factorio /factorio

# Install dependencies for mimalloc and huge pages
RUN DEBIAN_FRONTEND=noninteractive apt-get -qy install libhugetlbfs-bin cmake unzip --no-install-recommends

# Download and install mimalloc
RUN curl -L https://github.com/microsoft/mimalloc/archive/refs/tags/v1.7.1.zip -o mimalloc.zip \
    && unzip mimalloc.zip \
    && cd mimalloc-1.7.1 \
    && mkdir -p build \
    && cd build \
    && cmake .. \
    && make \
    && make install \
    && cd ../.. \
    && rm -rf mimalloc*

COPY files/*.sh /
COPY files/config.ini /opt/factorio/config/config.ini

VOLUME /factorio
EXPOSE $PORT/udp $RCON_PORT/tcp
ENTRYPOINT ["/docker-entrypoint.sh"]
