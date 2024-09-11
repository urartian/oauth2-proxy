# oauth2-proxy
oauth2-proxy with jwilder/nginx-proxy


# jwilder/nginx-proxy Overview

The `jwilder/nginx-proxy` is a popular Docker-based solution that automates Nginx reverse proxy configuration for Docker containers. It simplifies the process of setting up a reverse proxy for multiple containers by dynamically generating the required Nginx configuration based on environment variables. This allows different applications or services to be served from the same IP address, using different domain names or subdomains.

## Key Features of jwilder/nginx-proxy

### Dynamic Configuration
- The proxy configuration is automatically updated based on the Docker containers that start, stop, or restart. As new containers are launched, Nginx is reconfigured to route traffic appropriately.
- Containers are identified by setting environment variables like `VIRTUAL_HOST` to define the domain name.

### Multi-Domain and Subdomain Support
- You can route traffic to different containers based on domain names, subdomains, or even paths. For example, `app1.example.com` can point to one container, while `app2.example.com` points to another, all on the same server.

### SSL Support
- When combined with the companion container `jrcs/letsencrypt-nginx-proxy-companion`, `jwilder/nginx-proxy` can automatically generate and manage Let's Encrypt SSL certificates for your services. This allows for automatic HTTPS setup with minimal configuration.

### Environment Variables
- `VIRTUAL_HOST`: Define the domain or subdomain for the container (e.g., `VIRTUAL_HOST=app1.example.com`).
- `VIRTUAL_PORT`: (Optional) Specify the internal port to proxy to if it is different from the default (typically port 80).
- `VIRTUAL_PATH`: (Optional) Route traffic based on specific paths.

### Load Balancing
- If multiple containers share the same `VIRTUAL_HOST`, Nginx will automatically load balance the traffic between the containers.

### Zero Downtime
- The reverse proxy is updated in real-time without requiring a manual Nginx reload. This ensures that there is zero downtime when adding or removing services.

## How It Works
The reverse proxy works by listening for Docker events. When a new container with the appropriate environment variables (`VIRTUAL_HOST`, etc.) starts, it dynamically generates an Nginx configuration file for that container and reloads the proxy configuration. The reverse proxy routes requests based on the `Host` header to the correct container.

## Example Setup

### Starting the Nginx Reverse Proxy

```bash
docker run -d -p 80:80 \
    -v /var/run/docker.sock:/tmp/docker.sock:ro \
    --name nginx-proxy \
    jwilder/nginx-proxy
```

This starts the `jwilder/nginx-proxy` container and binds it to port 80. The container listens on the Docker socket (`/var/run/docker.sock`) for container events.

### Launching a Web Application with Virtual Host

For example, to start a web app and have it accessible at `app.example.com`:

```bash
docker run -d \
    -e VIRTUAL_HOST=app.example.com \
    --name web-app \
    my-web-app-image
```

The reverse proxy will automatically configure itself to route requests to `app.example.com` to the `web-app` container.

### Enabling HTTPS with Let's Encrypt

To enable HTTPS with automatic Let's Encrypt certificates, you can add the companion container:

```bash
docker run -d \
    --name nginx-proxy \
    -p 80:80 -p 443:443 \
    -v /etc/nginx/certs:/etc/nginx/certs:rw \
    -v /var/run/docker.sock:/tmp/docker.sock:ro \
    jwilder/nginx-proxy

docker run -d \
    --name nginx-proxy-companion \
    --volumes-from nginx-proxy \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    jrcs/letsencrypt-nginx-proxy-companion
```

Then, run your web app with additional environment variables to enable SSL:

```bash
docker run -d \
    -e VIRTUAL_HOST=app.example.com \
    -e LETSENCRYPT_HOST=app.example.com \
    -e LETSENCRYPT_EMAIL=your-email@example.com \
    --name web-app \
    my-web-app-image
```
