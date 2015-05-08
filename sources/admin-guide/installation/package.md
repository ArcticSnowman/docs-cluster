# Gluu-cluster deb package

## Gluu-cluster master deb package:

### Set this variable in root:

```
$ export MASTER_IPADDR = xxx.xxx.xxx.xxx
```

### Install docker:

Follow these instructions to install the package for Ubuntu Trusty 14.04 managed by docker.com:
[http://docs.docker.com/installation/ubuntulinux](http://docs.docker.com/installation/ubuntulinux/#docker-maintained-package-installation)

For the impatient, just type:

```
$ curl -sSL https://get.docker.com/ubuntu/ | sudo sh
```

#### Docker configuration:

```
$ echo "DOCKER_OPTS=\"-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock\"" >> /etc/default/docker
$ service docker restart
```

### Install salt-master:

```
$ echo deb http://ppa.launchpad.net/saltstack/salt/ubuntu `lsb_release -sc` main | sudo tee /etc/apt/sources.list.d/saltstack.list
$ wget -q -O- "http://keyserver.ubuntu.com:11371/pks/lookup?op=get&search=0x4759FA960E27C0A6" | sudo apt-key add -
$ apt-get update
$ apt-get install -y salt-master
```

### Install salt-minion:

```
$ apt-get install -y salt-minion
```

### Configure salt-minion

```
$ echo "master: $MASTER_IPADDR" >> /etc/salt/minion
```

### Install weave:

```
$ wget -O /usr/local/bin/weave https://github.com/weaveworks/weave/releases/download/latest_release/weave
$ chmod a+x /usr/local/bin/weave
```

NOTE: we can host weave binary in our gluu server.

Setup weave:

```
$ weave setup
```

Start weave

```
$ weave launch
```

### Prometheus:

Put `prometheus.conf` file in `/etc/gluu/prometheus/prometheus.conf`
(maybe there is a better place to put this file?).

Default `prometheus.conf` file:

```
# Global default settings.
global {
  scrape_interval: "15s"     # By default, scrape targets every 15 seconds.
  evaluation_interval: "15s" # By default, evaluate rules every 15 seconds.

  # Attach these extra labels to all timeseries collected by this Prometheus instance.
  labels: {
    label: {
      name: "monitor"
      value: "codelab-monitor"
    }
  }

  # Load and evaluate rules in this file every 'evaluation_interval' seconds. This field may be repeated.
  #rule_file: "prometheus.rules"
}

# A job definition containing exactly one endpoint to scrape: Here it's prometheus itself.
job: {
  # The job name is added as a label `job={job-name}` to any timeseries scraped from this job.
  name: "prometheus"
  # Override the global default and scrape targets from this job every 5 seconds.
  scrape_interval: "5s"

  # Let's define a group of targets to scrape for this job. In this case, only one.
  target_group: {
    # These endpoints are scraped via HTTP.
    target: "http://localhost:9090/metrics"
  }
}
```

#### Pull prometheus:

```
$ docker pull prom/prometheus
```

#### Run prometheus:

```
# docker run -d -v /etc/gluu/prometheus/prometheus.conf:/prometheus.conf --cidfile="/var/run/prometheus.cid" prom/prometheus
```

### Install gluu-flask:

#### Ubuntu packages

```
$ apt-get install libssl-dev python-dev swig
$ apt-get build-dep openssl
```

#### Install pip and virtualenv:

```
$ wget -q -O- https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py | python -
$ wget -q -O- https://raw.github.com/pypa/pip/master/contrib/get-pip.py | python -
$ export PATH="/usr/local/bin:$PATH"
$ pip install virtualenv
```

#### Clone gluu-flask:

```
$ git clone git@github.com:GluuFederation/gluu-flask.git /opt/gluu-flask
```

#### Install gluu-flask requirements:

```
$ virtualenv /opt/gluu-flask/env
$ /opt/gluu-flask/env/bin/pip install -r requirements.txt
```

## Gluu-cluster consumer deb package:

### Set this variable in root:

```
$ export MASTER_HOST_IP = xxx.xxx.xxx.xxx
```

### Install docker:

Follow these instructions to install the package for Ubuntu Trusty 14.04 managed by docker.com:
[http://docs.docker.com/installation/ubuntulinux](http://docs.docker.com/installation/ubuntulinux/#docker-maintained-package-installation)

For the impatient, just type:

```
$ curl -sSL https://get.docker.com/ubuntu/ | sudo sh
```

#### docker configuration:

```
$ echo "DOCKER_OPTS=\"-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock\"" >> /etc/default/docker
$ service docker restart
```

### Install salt-minion:

```
$ echo deb http://ppa.launchpad.net/saltstack/salt/ubuntu `lsb_release -sc` main | sudo tee /etc/apt/sources.list.d/saltstack.list
$ wget -q -O- "http://keyserver.ubuntu.com:11371/pks/lookup?op=get&search=0x4759FA960E27C0A6" | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install -y salt-minion
```

#### Configuration:

```
$ echo "master: $MASTER_HOST_IP" >> /etc/salt/minion
```

### Install weave:

```
$ sudo wget -O /usr/local/bin/weave https://github.com/weaveworks/weave/releases/download/latest_release/weave
$ sudo chmod a+x /usr/local/bin/weave
```

NOTE: we can host weave binary in our gluu server.

Setup weave:

```
$ weave setup
```

Start weave:

```
$ weave launch $MASTER_HOST_IP
```
