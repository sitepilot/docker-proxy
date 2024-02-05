FROM ghcr.io/sitepilot/runtime:v1 AS proxy

ARG TARGETARCH
ARG DOCKERGEN_VERSION=0.10.6

USER root

ENV PROXY_SSL_EMAIL="internal" \
    CADDY_DATA_DIR="$RUNTIME_WORKDIR/data"

RUN rt-install gnupg \
    && curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg \
    && curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-stable.list  \
    && rt-install caddy \
    && rt-rchown /etc/caddy

RUN wget -O - https://github.com/nginx-proxy/docker-gen/releases/download/$DOCKERGEN_VERSION/docker-gen-linux-$TARGETARCH-$DOCKERGEN_VERSION.tar.gz | tar xzf - -C /usr/local/bin

COPY --chown=$RUNTIME_UID:$RUNTIME_GID rootfs /

EXPOSE 80
EXPOSE 443

CMD ["/proxy"]
