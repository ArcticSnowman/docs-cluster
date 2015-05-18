## Gluu Cluster Packages

There are 3 main packages used by Gluu Cluster:

* Gluu Cluster Master or `gluu-master`
* Gluu Cluster API or `gluu-flask`
* Gluu Cluster Consumer or `gluu-consumer`

Things to remember:

* `gluu-flask` won't work without `gluu-master`, hence these 2 packages are mandatory and must be installed in a same host.
* In single server setup, `gluu-consumer` is not required at all.

### Gluu Cluster Master

Behind the scene, `gluu-master` consists of various components:

* [docker](https://www.docker.com/)
* [salt-master](https://github.com/saltstack/salt)
* [salt-minion](https://github.com/saltstack/salt)
* [weave](http://weave.works/)
* [prometheus](http://prometheus.io/)

#### Installing Gluu Cluster Master Components

To minimize the complexity of installation, those components are packaged into a single `deb` package called `gluu-master`.

The following snippet will install `gluu-master` package from Gluu repository:

```
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list
curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
apt-get install -y gluu-master
```

#### Configuring Gluu Cluster Master Components

##### docker

Configure ``docker`` daemon:

```
echo "DOCKER_OPTS=\"-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock\"" >> /etc/default/docker
service docker restart
```

##### salt-master

Set this variable in root:

```
export MASTER_IPADDR=xxx.xxx.xxx.xxx
```

where `xxx.xxx.xxx.xxx` is the IP address of the machine.

##### salt-minion

Configure `salt-minion`:

```
echo "master: $MASTER_IPADDR" >> /etc/salt/minion
service salt-minion restart
```

Register minion to master:

```
salt-key -y -a `hostname --long`
```

Test that minion is properly connected to master:

```
salt `hostname --long` test.ping
```

##### weave

Setup and run `weave`:

```
weave setup
weave launch
```

##### Prometheus

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

Run `prometheus`:

```
docker run -d -v /etc/gluu/prometheus/prometheus.conf:/etc/prometheus/prometheus.conf --cidfile="/var/run/prometheus.cid" prom/prometheus
```

The command above will pull prometheus image when necessary.

#### Registering Gluu Cluster Master as Provider

The master key will be registered when [Provider](../../reference/api/provider.md) entity is created.
Please note, this process requires `gluu-flask` installed and running.

### Gluu Cluster API

Install `gluu-flask` package:

```
apt-get install -y gluu-flask
```

This will install an executable called `gluuapi`.

Run `gluuapi`:

```
API_ENV=prod SALT_MASTER_IPADDR=$MASTER_IPADDR gluuapi
```

### Gluu Cluster Consumer

Behind the scene, `gluu-consumer` consists of various components:

* [docker](https://www.docker.com/)
* [salt-minion](https://github.com/saltstack/salt)
* [weave](http://weave.works/)

#### Installing Gluu Cluster Consumer Components

To minimize the complexity of installation, those components are packaged into a single `deb` package called `gluu-consumer`.

The following snippet will install `gluu-consumer` package from Gluu repository:

```
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list
curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
apt-get install -y gluu-consumer
```

Set this variable in root:

```
export MASTER_IPADDR=xxx.xxx.xxx.xxx
```

The `MASTER_IPADDR` must equal to `MASTER_IPADDR` mentioned in Gluu Cluster Master section.

##### docker

Configure `docker` daemon:

```
echo "DOCKER_OPTS=\"-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock\"" >> /etc/default/docker
service docker restart
```

##### salt-minion

Configure `salt-minion`:

```
echo "master: $MASTER_IPADDR" >> /etc/salt/minion
service salt-minion restart
```

##### weave

Setup and run `weave`:

```
weave setup
weave launch $MASTER_IPADDR
```

#### Registering Gluu Cluster Consumer as Provider

The consumer key will be registered when [Provider](../../reference/api/provider.md) entity is created.
Please note, this process requires `gluu-flask` installed and running.
