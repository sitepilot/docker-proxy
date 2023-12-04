# Docker Proxy

Docker Proxy is an automated Nginx reverse proxy for Docker which can proxy and load-balance HTTP and TCP/UDP traffic to
containers.

This image is based on our [Docker Runtime](https://github.com/sitepilot/docker-runtime) image, an optimized and
extensible Ubuntu container image.

## Usage

The containers being proxied must be in the same network and expose the port to be proxied, either using the `EXPOSE`
directive in their `Dockerfile` or with the `--expose` flag when running the `docker run` or `docker create` command. To
proxy traffic to a container, add `PROXY_HTTP_HOST` or `PROXY_STREAM_PORT` to the container's environment.

When your container exposes only one port, nginx-proxy will default to that port; otherwise, it defaults to port 80. If
you need to specify a different port, you can set either the `PROXY_HTTP_CONTAINER_PORT`
or `PROXY_STREAM_CONTAINER_PORT`
environment variable to select an alternative one. Note that this variable cannot be set to more than one port. A
container can expose one HTTP and one TCP/UDP port through the proxy.

### HTTP Proxy

Start the proxy container with the following command:

```bash
docker run -d -p 80:80 -p 443:443 -v /var/run/docker.sock:/tmp/docker.sock:ro ghcr.io/sitepilot/proxy:v1
```

Then, start any containers you want to be proxied with the `PROXY_HTTP_HOST` environment variable:

```bash
docker run -e PROXY_HTTP_HOST=subdomain.example.com  ...
```

If you start more containers with the same `PROXY_HTTP_HOST` environment variable, the proxy will load-balance traffic
between these containers.

### TCP/UDP Proxy (stream)

Start the proxy container with the following command:

```bash
docker run -d -p 8080:8080 -v /var/run/docker.sock:/tmp/docker.sock:ro ghcr.io/sitepilot/proxy:v1
```

Then, start any containers you want to be proxied with the `PROXY_STREAM_PORT` environment variable:

```bash
docker run -e PROXY_STREAM_PORT=8080  ...
```

If you start more containers with the same `PROXY_STREAM_PORT` environment variable, the proxy will load-balance traffic
between these containers.

### Additional upstream servers

When a container is deployed to multiple servers, you can specify additional upstream servers with
the `PROXY_HTTP_SERVERS` and `PROXY_STREAM_SERVERS` environment variable. The proxy will include these servers in the
upstream configuration and load-balance traffic among them.

## Proxy Configuration

The following environment variables are available to modify the configuration of the proxy container:

| Name                      | Value             |
|---------------------------|-------------------|
| `NGINX_HTTP_PORT`         | `80`              |
| `NGINX_HTTPS_PORT`        | `443`             |
| `NGINX_GZIP_ENABLED`      | `true`            |
| `NGINX_GZIP_COMP_LEVEL`   | `5`               |
| `NGINX_GZIP_VARY`         | `on`              |
| `NGINX_GZIP_PROXIED`      | `any`             |
| `NGINX_GZIP_STATIC`       | `off`             |
| `NGINX_SET_REAL_IP_FROM`  | `0.0.0.0/0`       |
| `NGINX_REAL_IP_HEADER`    | `X-Forwarded-For` |
| `NGINX_REAL_IP_RECURSIVE` | `on`              |
