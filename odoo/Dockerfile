# check=skip=SecretsUsedInArgOrEnv
ARG BUILD_FROM=odoo:18.0

FROM ${BUILD_FROM}

ARG BUILD_ARCH=amd64

# Switch to root for installation
USER root

# Container Environment variables
ENV \
    DEBIAN_FRONTEND="noninteractive" \
    HOME="/root" \
    LANG="C.UTF-8" \
    PS1="$(whoami)@$(hostname):$(pwd)$ " \
    S6_BEHAVIOUR_IF_STAGE2_FAILS=2 \
    S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0 \
    S6_CMD_WAIT_FOR_SERVICES=1 \
    S6_SERVICES_GRACETIME=0 \
    TERM="xterm-256color"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# PostgreSQL environment variables
ENV \
    PG_MAJOR=16 \
    PGDATA="/data/postgres" \
    POSTGRES_DB="odoo" \
    POSTGRES_USER="odoo" \
    POSTGRES_PASSWORD="odoo" \
    POSTGRES_INITDB_ARGS="--encoding=UTF8 --locale=en_US.UTF-8"

# Odoo environment variables
ENV \
    ODOO_RC="/etc/odoo/odoo.conf" \
    ODOO_DATA_DIR="/data/odoo"

# renovate: datasource=github-releases packageName=hassio-addons/bashio
ARG BASHIO_VERSION="v0.17.5"
# renovate: datasource=github-releases packageName=home-assistant/tempio
ARG TEMPIO_VERSION="2024.11.2"
# renovate: datasource=github-releases packageName=just-containers/s6-overlay
ARG S6_OVERLAY_VERSION="3.2.0.2"

#------- Install s6-overlay -------#
RUN \
    case "${BUILD_ARCH}" in \
        amd64) S6_ARCH="x86_64" ;; \
        aarch64) S6_ARCH="aarch64" ;; \
        *) echo "Unsupported arch: ${BUILD_ARCH}" && exit 1 ;; \
    esac \
    && curl -L -s -o /tmp/s6-overlay-noarch.tar.xz \
        "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-noarch.tar.xz" \
    && tar -C / -Jxpf /tmp/s6-overlay-noarch.tar.xz \
    && curl -L -s -o /tmp/s6-overlay-arch.tar.xz \
        "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-${S6_ARCH}.tar.xz" \
    && tar -C / -Jxpf /tmp/s6-overlay-arch.tar.xz \
    && rm -f /tmp/s6-overlay-*.tar.xz

#------- Install bashio, tempio and system dependencies -------#
RUN \
    apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        jq \
        tzdata \
        gnupg2 \
        lsb-release \
        locales \
        libnss-wrapper \
    \
    && curl -J -L -o /tmp/bashio.tar.gz \
        "https://github.com/hassio-addons/bashio/archive/${BASHIO_VERSION}.tar.gz" \
    && mkdir /tmp/bashio \
    && tar zxvf /tmp/bashio.tar.gz --strip 1 -C /tmp/bashio \
    && mv /tmp/bashio/lib /usr/lib/bashio \
    && ln -s /usr/lib/bashio/bashio /usr/bin/bashio \
    \
    && curl -L -s -o /usr/bin/tempio \
        "https://github.com/home-assistant/tempio/releases/download/${TEMPIO_VERSION}/tempio_${BUILD_ARCH}" \
    && chmod a+x /usr/bin/tempio \
    \
    && rm -fr /tmp/* /var/{cache,log}/* /var/lib/apt/lists/*

#------- Generate locale -------#
RUN \
    echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen \
    && locale-gen \
    && update-locale LANG=en_US.UTF-8

ENV LANG=en_US.UTF-8

#------- Install PostgreSQL 16 (no pgvector - edge host) -------#
RUN \
    curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc \
        | gpg --dearmor -o /usr/share/keyrings/postgresql-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/postgresql-keyring.gpg] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" \
        > /etc/apt/sources.list.d/pgdg.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        postgresql-${PG_MAJOR} \
        postgresql-client-${PG_MAJOR} \
    && rm -rf /var/lib/apt/lists/*

# Setup postgres user and directories
RUN \
    install --verbose --directory --owner postgres --group postgres --mode 1777 /var/lib/postgresql \
    && install --verbose --directory --owner postgres --group postgres --mode 3777 /var/run/postgresql \
    && mkdir -p /docker-entrypoint-initdb.d

ENV PATH="$PATH:/usr/lib/postgresql/$PG_MAJOR/bin"

# Configure PostgreSQL defaults
RUN \
    sed -ri "s!^#?(listen_addresses)\s*=\s*\S+.*!\1 = 'localhost'!" \
        /usr/share/postgresql/${PG_MAJOR}/postgresql.conf.sample

#------- Install Nginx -------#
RUN \
    apt-get update \
    && apt-get install -y --no-install-recommends nginx \
    && rm -rf /var/lib/apt/lists/* /etc/nginx/sites-enabled/* /etc/nginx/sites-available/*

#------- Install Python dependencies for Odoo addons -------#
RUN pip3 install --no-cache-dir --break-system-packages \
    websockets>=12.0

#------- Copy rootfs and config -------#
COPY rootfs/ /

# Create necessary directories
RUN \
    mkdir -p /data/odoo /data/postgres /data/logs/postgres /data/logs/nginx /data/logs/odoo \
    && mkdir -p /var/log/nginx \
    && chown -R odoo:odoo /data/odoo \
    && chown -R postgres:postgres /data/postgres /data/logs/postgres

# Set permissions on scripts
RUN find /etc/s6-overlay -name "run" -exec chmod +x {} \; 2>/dev/null || true

ENTRYPOINT ["/init"]

ARG BUILD_VERSION \
    BUILD_DATE \
    BUILD_DESCRIPTION \
    BUILD_NAME \
    BUILD_REF \
    BUILD_REPOSITORY

LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version="${BUILD_VERSION}" \
    maintainer="WOOWTECH <woowtech@designsmart.com.tw>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="WOOWTECH" \
    org.opencontainers.image.authors="WOOWTECH <woowtech@designsmart.com.tw>" \
    org.opencontainers.image.licenses="LGPL-3.0" \
    org.opencontainers.image.url="https://github.com/WOOWTECH" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
