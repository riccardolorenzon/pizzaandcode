---
title: "Dockerize your aiohttp app production ready with nginx and gunicorn"
date: 2020-11-15T12:51:19+01:00
disqus: false
---
One of the main purposes of using Docker is to make the production builds easy to package and ship without the overhead of additional configurations. 
The more configurations necessary to deploy an application, the more likely the whole CD process is gonna break anytime, Docker is a great help on that.

I have been working with several web frameworks, mostly for Python, usually the deployment process consists of a reverse proxy, `nginx` or `apache`, set in front of the application, that eventually forwards requests to an application server, like `uwsgi` or `gunicorn` for example. 

A reverse proxy is not strictly necessary to run an application, but it provides several advantages: 
   - Serving static content
   - Simplify the load balancer configuration
   - Act eventually as cache or layer in front of the application to prevent ddos or other attacks

The framework i'm currently using, `aiohttp`, supports several application servers, as described in the [documentation](https://docs.aiohttp.org/en/stable/deployment.html). 

On the design of this solution, i decided to use `gunicorn`, in order to semplify the `nginx` configuration file, and allow to scale the number of workers without further configurations.
Both `nginx` and `gunicorn` are running on the same docker container, the execution is managed by `supervisord`. 
Supervisor is a client/server system that allows its users to monitor and control a number of processes on UNIX-like operating systems.

Let's start by creating the aiohttp app:

```
python3 -m venv venv
source venv/bin/activate
pip install aiohttp cchardet aiodns gunicorn 
pip freeze > requirements.txt
```

The app is a simple aiohttp application with a single endpoint that returns a string: 
```
from aiohttp import web

async def handle(request):
    return web.Response(text='hello from aiohttp')

app = web.Application()
app.add_routes([web.get('/', handle)])

if __name__ == '__main__':
    web.run_app(app)
```

Afterwards we test everything is working correctly:
```
python app.py
curl localhost:8080/
```

```
hello from aiohttp
```

Our application is working as expected, it's now time to package and make it production ready.


`nginx.conf`:
```
worker_processes 1;
user nobody nogroup;
events {
    worker_connections 1024;
}
http {
    ## Main Server Block
    server {
        ## Open by default.
        listen                80 default_server;
        server_name           main;
        client_max_body_size  200M;

        ## Main site location.
        location / {
            proxy_pass                          http://127.0.0.1:8080;
            proxy_set_header                    Host $host;
            proxy_set_header X-Forwarded-Host   $server_name;
            proxy_set_header X-Real-IP          $remote_addr;
        }
    }
}
```

`supervisord.conf`

``` 
[supervisord]
nodaemon=true

[program:nginx]
command=/usr/sbin/nginx -g "daemon off;"
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
# Graceful stop, see http://nginx.org/en/docs/control.html
stopsignal=QUIT

[program:gunicorn]
command=gunicorn app:app -c /app/gunicorn_conf.py
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

`entrypoint.sh`:
```
#!/usr/bin/env sh
set -e

# Get the maximum upload file size for Nginx, default to 0: unlimited
USE_NGINX_MAX_UPLOAD=${NGINX_MAX_UPLOAD:-0}

# Get the number of workers for Nginx, default to 1
USE_NGINX_WORKER_PROCESSES=${NGINX_WORKER_PROCESSES:-1}

# Set the max number of connections per worker for Nginx, if requested 
# Cannot exceed worker_rlimit_nofile, see NGINX_WORKER_OPEN_FILES below
NGINX_WORKER_CONNECTIONS=${NGINX_WORKER_CONNECTIONS:-1024}

# Get the listen port for Nginx, default to 80
USE_LISTEN_PORT=${LISTEN_PORT:-80}

cp /app/nginx.conf /etc/nginx/nginx.conf

# For Alpine:
# Explicitly add installed Python packages and uWSGI Python packages to PYTHONPATH
# Otherwise uWSGI can't import Flask
if [ -n "$ALPINEPYTHON" ] ; then
    export PYTHONPATH=$PYTHONPATH:/usr/local/lib/$ALPINEPYTHON/site-packages:/usr/lib/$ALPINEPYTHON/site-packages
fi

exec "$@"
```

`start.sh`:
```
#! /usr/bin/env sh
set -e

# If there's a prestart.sh script in the /app directory, run it before starting
PRE_START_PATH=/app/prestart.sh
echo "Checking for script in $PRE_START_PATH"
if [ -f $PRE_START_PATH ] ; then
    echo "Running script $PRE_START_PATH"
    . $PRE_START_PATH
else
    echo "There is no script $PRE_START_PATH"
fi

# Start Supervisor, with Nginx and uWSGI
exec /usr/bin/supervisord
```

`Dockerfile`: 
```
FROM python:3.8

LABEL maintainer="riccardo lorenzon <riccardo.lorenzon@gmail.com" 

RUN apt-get update && apt-get install -y nginx supervisor \
&& rm -rf /var/lib/apt/lists/*

RUN rm /etc/nginx/sites-available/default 
ADD ./ /app

RUN pip3 install -r /app/requirements.txt

# By default, Nginx will run a single worker process, setting it to auto
# will create a worker for each CPU core
ENV NGINX_WORKER_PROCESSES 1

# Custom Supervisord config
COPY supervisord-debian.conf /etc/supervisor/conf.d/supervisord.conf

RUN chmod +x /app/start.sh

# Copy the entrypoint that will generate Nginx additional configs
RUN chmod +x /app/entrypoint.sh

ENTRYPOINT ["/app/entrypoint.sh"]

WORKDIR /app
CMD ["/app/start.sh"]
```


