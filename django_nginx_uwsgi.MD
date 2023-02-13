# Django Uwsgi & Nginx

## **Setting up Django and your web server with uWSGI and nginx**

### Install UWsgi

```bash
sudo apt install uwsgi uwsgi-plugin-python3 gcc
```

Create a file in `/etc/uwsgi/apps-available/` with the name of your project. For example `tusd.ini` and add the following content:

```ini
[uwsgi]
# Base directory for the project
project = server
uid = User
base = /home/projects/tusd
# Chdir to the base directory
chdir = %(base)
# Set the python path to the base directory
home = %(base)/env
# Set WSGI module to the project's wsgi.py
module = %(project).wsgi:application
# Enable the server to reload itself when code changes
env = DJANGO_SETTINGS_MODULE=server.settings
# Master process Enabled
master = true
# Process name
pidfile=/tmp/tusd.pid
# Number of worker processes
processes = 5
# Socket file
socket = %(base)/run/tusd.sock
# Set permissions for the socket file
chown-socket = %(uid):www-data
# Set permissions for the socket file
chmod-socket = 660
# Maximum requests a worker will process before automatically restarting
max-requests=5000
# Clear environment on exit
vacuum = true
# Enable threads
enable-threads  = true
# Set the log file
daemonize=%(base)/logs/uwsgi.log
# Set Max log size
log-maxsize = 20971520
# Set log rotation
log-backupname = %(base)/logs/%n.log
# Serialize accept() usage (if possibie)
thunder-lock = true
# Reload uWSGI if the specified file or directory is modified/touched
touch-reload = %(base)
# Post-buffering (in bytes)
post-buffering = 204800
# Set the socket timeout
socket-timeout = 300
# Set the socket queue size
reload-mercy = 8
# Worker reload on as
reload-on-as = 512
# Set the max worker lifetime
max-worker-lifetime = 300
# Respawn workers when memory limit is exceeded
harakiri = 50
# Disable logging
disable-logging = True
# Set interpreter to single mode (for python)
single-interpreter = true
```

### Create the log directory

```bash
sudo mkdir /home/projects/tusd/logs
```

### Create the run directory

```bash
sudo mkdir /home/projects/tusd/run
```

### Enable the app

```bash
sudo ln -s /etc/uwsgi/apps-available/tusd.ini /etc/uwsgi/apps-enabled/tusd.ini
```

### Start the app

```bash
sudo systemctl start uwsgi
```

### Check the status

```bash
sudo systemctl status uwsgi
```

## Install Nginx

```bash
sudo apt install nginx
```

Create a file in `/etc/nginx/sites-available/` with the name of your project. For example `tusd` and add the following content:

```nginx
upstream tusd {
    server unix:///home/projects/tusd/run/tusd.sock;
}

server {
    listen 80;
    server_name localhost;
    charset utf-8;
    client_max_body_size 75M;

    #ssl_certificate_key  /etc/letsencrypt/live/localhost/privkey.pem;
    #ssl_certificate  /etc/letsencrypt/live/localhost/fullchain.pem;
    #ssl_dhparam /etc/nginx/dhparams/dhparams.pem;
    #ssl_protocols TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
    #ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;    
    #ssl_ecdh_curve secp521r1:secp384r1;
    #ssl_prefer_server_ciphers on;

    gzip on;
    access_log /home/projects/tusd/logs/nginx-access.log;
    error_log /home/projects/tusd/logs/nginx-error.log;

    location / {
        uwsgi_pass tusd;
        include /etc/nginx/uwsgi_params;
    }

    location /staticfiles/ {
       alias /home/projects/tusd/staticfiles/;
    }

    location /media/ {
        alias /home/projects/tusd/media/;
    }
}
```

### Enable the site

```bash
sudo ln -s /etc/nginx/sites-available/tusd /etc/nginx/sites-enabled/tusd
```

### Start the server

```bash
sudo systemctl start nginx
```

### you need to change the permissions of the socket file

```bash
sudo chown:www-data /home/projects/tusd/run/tusd.sock
```

### Reload the server

```bash
sudo systemctl reload nginx
```

## References

* [Django Uwsgi & Nginx](https://www.devdungeon.com/content/how-deploy-django-nginx-and-uwsgi)
* [Nginx](https://www.nginx.com/resources/wiki/start/topics/examples/full/)
* [Uwsgi](https://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html)
