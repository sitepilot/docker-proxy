FROM ghcr.io/sitepilot/runtime:v1 AS proxy

ARG TARGETARCH
ARG DOCKERGEN_VERSION=0.10.6

USER root

ENV NGINX_LOGS_DIR="$RUNTIME_WORKDIR/logs" \
    NGINX_CERTS_DIR="$RUNTIME_WORKDIR/certs"

RUN rt-install nginx libnginx-mod-stream \
    && rm /etc/nginx/sites-enabled/default \
    && mkdir -p /etc/nginx/streams-enabled \
    && rt-rchown /etc/nginx /var/log/nginx /var/lib/nginx \
    && setcap "cap_net_bind_service=+ep" /usr/sbin/nginx

RUN wget -O - https://github.com/nginx-proxy/docker-gen/releases/download/$DOCKERGEN_VERSION/docker-gen-linux-$TARGETARCH-$DOCKERGEN_VERSION.tar.gz | tar xzf - -C /usr/local/bin

COPY --chown=$RUNTIME_UID:$RUNTIME_GID rootfs /

CMD ["/proxy"]
