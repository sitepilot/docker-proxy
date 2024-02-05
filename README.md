# Docker Proxy

Docker Proxy is an automated Caddy reverse proxy for Docker which can proxy and load-balance HTTP traffic to
containers and automatically issue SSL-certificates.

This image is based on our [Docker Runtime](https://github.com/sitepilot/docker-runtime) image, an optimized and
extensible Ubuntu container image.

## Usage

The containers being proxied must be in the same network and expose the port to be proxied, either using the `EXPOSE`
directive in their `Dockerfile` or with the `--expose` flag when running the `docker run` or `docker create` command. To
proxy traffic to a container, add `PROXY_HOST` to the container's environment.

When your container exposes only one port, the proxy will default to that port; otherwise, it defaults to port 80. If
you need to specify a different port, you can set the `PROXY_PORT` environment variable to select an alternative one.
Note that this variable cannot be set to more than one port. A
container can expose one HTTP port through the proxy.

### Getting Started

Start the proxy container with the following command:

```bash
docker run -d --rm \
  -p 80:80 -p 443:443 \
  -v proxy_data:/app/data \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  ghcr.io/sitepilot/proxy:latest
```

Then, start any containers you want to be proxied with the `PROXY_HOST` environment variable:

```bash
docker run -e PROXY_HOST=subdomain.example.com  ...
```

If you start more containers with the same `PROXY_HOST` environment variable, the proxy will load-balance traffic
between these containers.

### Automatic SSL

By default, a self-signed certificate is used. If you would like to enable automatic SSL-certificate issuing, you need
to set the `PROXY_SSL_EMAIL` environment variable to a valid email address on the container.

```bash
docker run \
  -e PROXY_SSL_EMAIL=letsencrypt@example.com \
  -e PROXY_HOST=subdomain.example.com \
  ...
```

To enable automatic HTTPS for all proxied containers set the `PROXY_SSL_EMAIL` environment variable
on the proxy container itself.

### Custom SSL-certificate

To use a custom SSL-certificate, specify the `PROXY_SSL_KEY_FILE` and `PROXY_SSL_CERT_FILE` environment
variables on the container.

```bash
docker run \
  -e PROXY_SSL_KEY_FILE=/certs/example.com.key \
  -e PROXY_SSL_CERT_FILE=/certs/example.com.cert \
  -e PROXY_HOST=example.com \
  ...
```

⚠️ Ensure that both files exist in the proxy container.

To enable a custom (wildcard) SSL-certificate for all proxied containers set the `PROXY_SSL_KEY_FILE`
and `PROXY_SSL_CERT_FILE` environment variables on the proxy container itself.

### HTTP Basic Authentication

To enable HTTP Basic Authentication, specify the `PROXY_BASIC_AUTH` environment variable on the container.

```bash
docker run \
  -e PROXY_BASIC_AUTH='Bob $2a$14$Zkx19XLiW6VYouLHR5NmfOFU0z2GTNmpkT/5qqR7hx4IjWJPDhjvG' \
  -e PROXY_HOST=example.com \
  ...
```

To enable HTTP Basic Authentication for all proxied containers set the `PROXY_BASIC_AUTH` environment variable
on the proxy container itself.

### Additional Servers

When deploying a container to multiple servers, you can specify additional upstream servers (comma-separated) using
the `PROXY_SERVERS` environment variable. The proxy will include these servers in the configuration and
load-balance traffic among them.

```bash
docker run \
  -e PROXY_SERVERS=1.2.3.4:80,1.2.3.5:80\
  -e PROXY_HOST=example.com \
  ...
```
