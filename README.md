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
you need to specify a different port, you can set the `PROXY_CONTAINER_PORT` environment variable to select an alternative one. Note that this variable cannot be set to more than one port. A
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

### Automatic HTTPS

By default, a self-signed certificate is used. If you would like to enable automatic HTTPS, you need to set the 
`PROXY_SSL_EMAIL` environment variable to a valid email address on the container.

```bash
docker run -e PROXY_SSL_EMAIL=letsencrypt@example.com -e PROXY_HOST=subdomain.example.com ...
```

### Custom SSL-certificate

To use a custom (wildcard) SSL certificate for all hosts, specify the following environment variables on the proxy container:

* `PROXY_SSL_KEY_PATH`: Path to the private key file.
* `PROXY_SSL_CERT_PATH`: Path to the certificate file.

Ensure that both paths are mounted to the proxy container.

```bash
docker run -d --rm \
  -p 80:80 -p 443:443 \
  -v proxy_data:/app/data \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v ./certs:/certs \
  -e PROXY_SSL_KEY_PATH=/certs/example.com.key \
  -e PROXY_SSL_CERT_PATH=/certs/example.com.cert \
  ghcr.io/sitepilot/proxy:latest
```

To use a custom SSL-certificate for a single host, specify the following environment variables on the container:

* `PROXY_SSL_KEY_PATH`: Path to the private key file in the proxy container.
* `PROXY_SSL_CERT_PATH`: Path to the certificate file in the proxy container.

Ensure that both paths are mounted to the proxy container.

```bash
docker run \
  -e PROXY_HOST=subdomain.example.com \
  -e PROXY_SSL_KEY_PATH=/certs/example.com.key \
  -e PROXY_SSL_CERT_PATH=/certs/example.com.crt \
  ...
```

### Additional Servers

When deploying a container to multiple servers, you can specify additional upstream servers (comma-separated) using 
the `PROXY_SERVERS` environment variable. The proxy will include these servers in the configuration and 
load-balance traffic among them.
