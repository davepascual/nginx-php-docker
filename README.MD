# NGINX and PHP Docker Setup

## Prerequisite

### Running PHP app's image.

Build the PHP app image first in `php` folder.

```console
docker build -t php-app .
```

Run the build image with environt variable `app_name`.
> By default `php-app`'s EXPOSE port is `8000`

```console
docker run -d -p <PORT:8000> -e="app_name=<APP_NAME>" --name <CONTAINER_NAME> php-app
```

*example:*
```console
docker run -d -p <8001:8000> -e="app_name=AppOne" --name <php-app-one> php-app
```

You can run multiple `php-app` image as container with different ports, container name and environment variable `app_name` value.

### Running NGINX webserver

Running NGINX service.
```console
docker compose -d up
```

### NGINX configuration

#### Setting up reverse proxy

Setup NGINX' configuration in `./nginx/default.conf`.

```nginxconf
server {
    # Listening port
    # When a request with port 81 receieves it will reverse proxy the local machine's URL
    listen 81;
    # Root URL path
    location / {
        # Reverse proxy
        proxy_pass http://host.docker.internal:<PORT>;
    }
}
```

After setting up NGINX' configuration, add the expose port in `./nginx/docker-compose.yaml`

```yaml
services:
  webserver:
    image: nginx:latest
    volumes:
      # Mount local configuration to containers' configuration
      - ./default.conf:/etc/nginx/conf.d/default.conf
    ports:
      # Default nginx
      - 80:80
      # PHP app
      - 81:81
```

Then acccess it by [http://localhost:81](http://localhost:81) .

> You can add mutltiple PHP application using reverse proxy. Just remember to include the new added the listen port of new server configuration to `docker-compose.yaml` ports.

#### Setting up upstream, Load Balancer

Using load balancer, you must have run multiple php app container.

Add different servers, php app container.

```nginxconf
# Load balancing with php applications different servers
upstream php-app{
    server host.docker.internal:8000;
}

# PHP Load balancer in PORT 8009
server {
    listen 8009;
    location / {
        proxy_pass http://php-app;
    }
}
```

Then acccess it by [http://localhost:8009](http://localhost:8009) .

Refresh page and see the difference of value of `app_name` environment variable.


> Add expose port 8009 to `docker-compose.yaml` ports.
