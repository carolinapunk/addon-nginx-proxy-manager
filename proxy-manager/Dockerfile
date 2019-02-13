ARG BUILD_FROM=hassioaddons/base:2.3.1
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Setup base
# hadolint ignore=DL3003
RUN \
    apk add --no-cache --virtual .build-dependencies \
        build-base=0.5-r1 \
        git=2.18.1-r0 \
        npm=8.14.0-r0 \
        patch=2.7.6-r2 \
        python2=2.7.15-r1 \
        yarn=1.7.0-r0 \
    \
    && apk add --no-cache \
        apache2-utils=2.4.38-r3 \
        certbot=0.25.1-r1 \
        mariadb-client=10.2.19-r1 \
        mariadb=10.2.19-r1 \
        nginx-mod-stream=1.14.2-r0 \
        nginx=1.14.2-r0 \
        nodejs=8.14.0-r0 \
        openssl=1.0.2q-r0 \
        dnsmasq=2.79-r3 \
        libcap=2.25-r1 \
    \
    && curl -J -L -o /tmp/nginxproxymanager.tar.gz \
        "https://github.com/jc21/nginx-proxy-manager/archive/2.0.8.tar.gz" \
    \
    && curl -s -f -L \
        "https://install.goreleaser.com/github.com/tj/node-prune.sh" \
            | bash -s -- -b /tmp \
    \
    && mkdir /app \
    && tar zxvf \
        /tmp/nginxproxymanager.tar.gz \
        --strip 1 -C /app \
    \
    && cd /app \
    && yarn install \
    && npm run-script build \
    && rm -rf node_modules \
    && yarn install --prod \
    && /tmp/node-prune \
    \
    && mkdir -p /opt/nginx-proxy-manager/src \
    && cp -r /app/dist /opt/nginx-proxy-manager/ \
    && cp -r /app/knexfile.js /opt/nginx-proxy-manager/ \
    && cp -r /app/node_modules /opt/nginx-proxy-manager/ \
    && cp -r /app/package.json /opt/nginx-proxy-manager/ \
    && cp -r /app/src/backend /opt/nginx-proxy-manager/src/ \
    \
    && sed -i 's#/data/logs/dead_host-{{ id }}.log#/proc/1/fd/1#g' \
        /opt/nginx-proxy-manager/src/backend/templates/dead_host.conf \
    && sed -i 's#/data/logs/redirection_host-{{ id }}.log#/proc/1/fd/1#g' \
        /opt/nginx-proxy-manager/src/backend/templates/redirection_host.conf \
    && sed -i 's#/data/logs/proxy_host-{{ id }}.log#/proc/1/fd/1#g' \
        /opt/nginx-proxy-manager/src/backend/templates/proxy_host.conf \
    \
    && mkdir -p \
        /run/mysqld \
        /run/nginx \
    \
    && rm -f -r \
        /app \
        /etc/init.d/nginx \
        /etc/logrotate.d/nginx \
        /etc/nginx \
        /opt/nginx-proxy-manager/node_modules/bcrypt/build \
        /tmp/.[!.]* \
        /tmp/* \
        /usr/lib/node_modules \
        /var/lib/mysql \
        /var/lib/nginx \
        /var/log/nginx \
        /var/tmp/nginx \
        /var/www \
    && apk del --purge .build-dependencies

# Copy root filesystem
COPY rootfs /

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_REF
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="Nginx Proxy Manager" \
    io.hass.description="Lorem ipsum" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Franck Nijhof <frenck@addons.community>" \
    org.label-schema.description="Lorem ipsum" \
    org.label-schema.build-date=${BUILD_DATE} \
    org.label-schema.name="Nginx Proxy Manager" \
    org.label-schema.schema-version="1.0" \
    org.label-schema.url="https://community.home-assistant.io/?u=frenck" \
    org.label-schema.usage="https://github.com/hassio-addons/addon-nginx-proxy-manager/tree/master/README.md" \
    org.label-schema.vcs-ref=${BUILD_REF} \
    org.label-schema.vcs-url="https://github.com/hassio-addons/addon-nginx-proxy-manager" \
    org.label-schema.vendor="Community Hass.io Add-ons"