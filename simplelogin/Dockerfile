ARG BUILD_FROM=ghcr.io/hassio-addons/ubuntu-base/amd64:6.1.2
# hadolint ignore=DL3006



FROM ${BUILD_FROM} as root


FROM  simplelogin/app:3.3.0 as sl

# Environment variables
ENV \
    DEBIAN_FRONTEND="noninteractive" \
    HOME="/root" \
    LANG="C.UTF-8" \
    PS1="$(whoami)@$(hostname):$(pwd)$ " \
    S6_BEHAVIOUR_IF_STAGE2_FAILS=2 \
    S6_CMD_WAIT_FOR_SERVICES=1 \
    TERM="xterm-256color"

ARG BUILD_ARCH=amd64

COPY --from=root /etc/cont-finish.d /etc/cont-finish.d
COPY --from=root /etc/cont-init.d /etc/cont-init.d
COPY --from=root /usr/bin/service /usr/bin/service

# Copy root filesystem
COPY rootfs /

RUN \
    apt-get update \
    \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        jq \
        tzdata \
    \
    && S6_ARCH="${BUILD_ARCH}" \
    && if [ "${BUILD_ARCH}" = "i386" ]; then S6_ARCH="x86"; fi \
    && if [ "${BUILD_ARCH}" = "armv7" ]; then S6_ARCH="arm"; fi \
    \
    && curl -L -s "https://github.com/just-containers/s6-overlay/releases/download/v2.1.0.2/s6-overlay-${S6_ARCH}.tar.gz" \
        | tar zxvf - -C / \
    \
    && mkdir -p /etc/fix-attrs.d \
    && mkdir -p /etc/services.d \
    \
    && curl -J -L -o /tmp/bashio.tar.gz \
        "https://github.com/hassio-addons/bashio/archive/v0.9.0.tar.gz" \
    && mkdir /tmp/bashio \
    && tar zxvf \
        /tmp/bashio.tar.gz \
        --strip 1 -C /tmp/bashio \
    \
    && mv /tmp/bashio/lib /usr/lib/bashio \
    && ln -s /usr/lib/bashio/bashio /usr/bin/bashio \
    \
    && rm -fr \
        /tmp/* \
        /var/{cache,log}/* \
        /var/lib/apt/lists/*

RUN apt-get update && apt-get --no-install-recommends -y install sudo postgresql postgresql-contrib      && rm -fr \
        /tmp/* \
        /var/{cache,log}/* \
        /var/lib/apt/lists/*

RUN echo postfix postfix/main_mailer_type string "'Internet Site'" | debconf-set-selections && \
    echo postfix postfix/mynetworks string "127.0.0.0/8" | debconf-set-selections && \
    echo postfix postfix/mailname string temporary.example.com | debconf-set-selections && \
    apt-get update -q -q && \
    apt-get --yes --force-yes --no-install-recommends install postfix postfix-pgsql     && rm -fr \
        /tmp/* \
        /var/{cache,log}/* \
        /var/lib/apt/lists/*

RUN apt-get update && apt-get --no-install-recommends install pwgen -y     && rm -fr \
        /tmp/* \
        /var/{cache,log}/* \
        /var/lib/apt/lists/*

# Entrypoint & CMD
ENTRYPOINT [ "/init" ]

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_REF
ARG BUILD_VERSION


# Labels
LABEL \
    io.hass.name="Test Server" \
    io.hass.description="Test Server " \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Elcajon" \
    org.opencontainers.image.title="Test Server" \
    org.opencontainers.image.description="Test Server" \
    org.opencontainers.image.vendor="Home Assistant Add-on Repository" \
    org.opencontainers.image.authors="Elcajon" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://github.com/elcajon/homeassistant-addons" \
    org.opencontainers.image.source="https://github.com/elcajon/homeassistant-addons" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
