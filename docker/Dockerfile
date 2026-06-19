# Factorio headless server image.
#
# Forked from factoriotools/factorio-docker (MIT, see LICENSE.factoriotools):
# the entrypoint and helper scripts under files/ and the rcon client are theirs,
# inherited verbatim so the env-var/volume contract stays identical. This
# Dockerfile is trimmed to linux/amd64 only (all our hosts are amd64), dropping
# their box64/ARM compatibility layer.

# build rcon client
FROM debian:stable-slim AS rcon-builder
RUN apt-get -q update \
    && DEBIAN_FRONTEND=noninteractive apt-get -qy install build-essential

WORKDIR /src
COPY rcon/ /src
RUN make

# build factorio image
FROM debian:stable-slim
LABEL maintainer="https://github.com/behindcurtain3/factorio-image"

ARG USER=factorio
ARG GROUP=factorio
ARG PUID=845
ARG PGID=845

# optionally utilize a built-in map-gen-preset (see data/base/prototypes/map-gen-presets)
ARG PRESET

# number of retries that curl will use when pulling the headless server tarball
ARG CURL_RETRIES=8

ENV PORT=34197 \
    RCON_PORT=27015 \
    SAVES=/factorio/saves \
    PRESET="$PRESET" \
    CONFIG=/factorio/config \
    MODS=/factorio/mods \
    SCENARIOS=/factorio/scenarios \
    SCRIPTOUTPUT=/factorio/script-output \
    PUID="$PUID" \
    PGID="$PGID" \
    DLC_SPACE_AGE="true"

SHELL ["/bin/bash", "-eo", "pipefail", "-c"]

RUN apt-get -q update \
    && DEBIAN_FRONTEND=noninteractive apt-get -qy install ca-certificates curl jq pwgen xz-utils procps gettext-base file --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

RUN groupadd --system --gid "$PGID" "$GROUP" \
    && useradd --system --uid "$PUID" --gid "$PGID" --no-create-home --shell /bin/sh "$USER"

# version checksum of the archive to download
ARG VERSION
ARG SHA256

LABEL factorio.version=${VERSION}

ENV VERSION=${VERSION} \
    SHA256=${SHA256}

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
    && curl -sSL "https://www.factorio.com/get-download/$VERSION/headless/linux64" -o "$archive" --retry $CURL_RETRIES \
    && echo "$SHA256  $archive" | sha256sum -c \
    || (sha256sum "$archive" && file "$archive" && exit 1) \
    && tar xf "$archive" --directory /opt \
    && chmod ugo=rwx /opt/factorio \
    && rm "$archive" \
    && ln -s "$SCENARIOS" /opt/factorio/scenarios \
    && ln -s "$SAVES" /opt/factorio/saves \
    && mkdir -p /opt/factorio/config/ \
    && chown -R "$USER":"$GROUP" /opt/factorio /factorio

COPY files/*.sh /
COPY files/config.ini /opt/factorio/config/config.ini
COPY --from=rcon-builder /src/rcon /bin/rcon

VOLUME /factorio
EXPOSE $PORT/udp $RCON_PORT/tcp
ENTRYPOINT ["/docker-entrypoint.sh"]
