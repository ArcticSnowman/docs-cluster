[TOC]
# Installation
The Gluu Cluster only supports Ubuntu for now. There is only one mandatory package need to be installed in a machine; the `gluu-engine` package.
There is one additional package, `gluu-cluster-webui`, that provides a user friendly way of using the API and managing the cluster. Note, this package must be installed in same host with `gluu-engine`.

## Prerequisites

### Minimum Linux Kernel

Cluster requires at least kernel 3.10 at minimum. We can check whether we're using supported kernel.

    uname -r

Please note, due to [issue with kernel 3.13.0-77](../known-issues#unsupported-kernel), this version should be avoided.

## Docker Engine and Docker Machine Packages

### Installing Docker Engine (Deprecated)

Run this:

```
$ sudo curl -fsSL https://raw.githubusercontent.com/GluuFederation/cluster-tools/master/get_docker.sh | sh
```

### Installing Docker Machine

Download the Docker Machine binary and extract it to our PATH:

```
curl -L https://github.com/docker/machine/releases/download/v0.7.0/docker-machine-`uname -s`-`uname -m` > /usr/local/bin/docker-machine && \
chmod +x /usr/local/bin/docker-machine
```

## Gluu Engine API

### Installing gluu-engine

First things first, we need to add Gluu repository and install few packages:

```sh
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list \
    && curl -s https://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
apt-get -y install git swig openjdk-7-jre-headless oxd-license-validator python-pip python-dev libssl-dev
```

Clone the `gluu-engine` project from our GitHub repository:

```sh
mkdir -p /root
cd /root
git clone https://github.com/GluuFederation/gluu-engine.git
```

Afterwards, we need to install the `gluuengine` Python package (latest stable release is `0.5.0-beta6`) and its dependencies:

```sh
cd gluu-engine
git checkout 0.5.0-beta6
mkdir -p /root/.virtualenvs
pip install virtualenv
virtualenv /root/.virtualenvs/gluu-engine
source /root/.virtualenvs/gluu-engine/bin/activate
pip install -U pip
pip install -r requirements.txt
python setup.py install
```

### Creating Data and Log Directories

All data and logs are saved under predefined directories. Create the directories if not exist:

```sh
mkdir -p /var/log/gluuengine  # directory for all gluuengine logs
mkdir -p /var/lib/gluuengine/db  # directory for gluuengine data
```

### Preparing Database

All data is saved into MongoDB, hence we need to prepare the database before running `gluu-engine`:

```sh
docker run -d --name mongo -v /var/lib/gluuengine/db/mongo:/data/db -p 27017:27017 mongo
```

### Running gluu-engine

The simplest way to run `gluu-engine` is to use `gunicorn` webserver.
But before doing it, we need to define how many workers `gunicorn` should use.
There's no exact formula about how many workers needed, but as `gunicorn` mentioned about
using CPU core number, we can use the following recipe to check the number:

```sh
python -c 'from multiprocessing import cpu_count; print (cpu_count() * 2) + 1'
```

For example, if we have 4 CPU cores, then we'll get `9` as the result returned from recipe above.

```sh
gunicorn -b 127.0.0.1:8080 --log-level warning --access-logfile - --error-logfile - -w 9 -k gthread -e API_ENV=prod 'gluuengine.app:create_app()'
```

Note, this will make the app running in foreground. To run `gluu-engine` app in background, we recommend `supervisor` to manage the process.

```sh
apt-get install -y supervisor
```

Add configuration below and save it as `/etc/supervisor/conf.d/gluuengine.conf` file:

```ini
[program:gluu-engine]
command=/root/.virtualenvs/gluu-engine/bin/gunicorn -b 127.0.0.1:8080 --log-level warning --access-logfile - --error-logfile - -w 9 -k gthread -e API_ENV=prod 'gluuengine.app:create_app()'
stdout_logfile=/var/log/gluuengine/api.log
stderr_logfile=/var/log/gluuengine/api.log
```

Each time we modify the config file above, we need to run `supervisorctl reload`.
Double-check whether `gluu-engine` is running:

```sh
supervisorctl status gluu-engine
# here's the output example: gluu-engine     RUNNING    pid 13025, uptime 0:00:21
```

## Gluu Cluster Web Interface package

### Installing gluu-cluster-webui

Run the following commands to install gluu-cluster-webui:
```
apt-get install -y gluu-cluster-webui
a2dissite 000-default
service apache2 restart
```

There are few things we need to know about Gluu Cluster Web UI:

1. The application is bind to 127.0.0.1 (localhost) as currently there's no protection mechanism yet.
2. The application is listening on port 8800 to avoid conflict with port used by nodes.

First things first, we need to add Gluu repository:

```
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list \
    && curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
```

Run the following commands to install the `gluu-cluster-webui` package:

```
apt-get install -y gluu-cluster-webui
```
