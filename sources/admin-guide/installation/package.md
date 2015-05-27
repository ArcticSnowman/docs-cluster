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

Download the `postinstall.py` script:

```
wget https://raw.githubusercontent.com/GluuFederation/gluu-cluster-postinstall/master/postinstall.py
```

Before running the script, make sure we already know which IP address we are going to use.
Afterwards, run the `postinstall.py` script and and answer all questions prompted by the script:

```
python postinstall.py
```

Here's an example of script's prompt:

```
root@gluu-master:~# python postinstall.py
Enter host type (ex. master or consumer) : master
Enter MASTER_IPADDR (ex xxx.xxx.xxx.xxx) : 128.199.242.74
```

Press enter to continue the process.

### Gluu Cluster API

In master host, install `gluu-flask` package:

```
apt-get install -y gluu-flask
```

This will install an executable called `gluuapi`.

Run `gluuapi`:

```
API_ENV=prod SALT_MASTER_IPADDR=128.199.242.74 nohup gluuapi &
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

#### Configuring Gluu Cluster Consumer Components

Download the `postinstall.py` script:

```
wget https://raw.githubusercontent.com/GluuFederation/gluu-cluster-postinstall/master/postinstall.py
```

Before running the script, make sure we already know the master's IP address we are going to use.
Afterwards, run the `postinstall.py` script and and answer all questions prompted by the script:

```
python postinstall.py
```

Here's an example of script's prompt:

```
root@gluu-master:~# python postinstall.py
Enter host type (ex. master or consumer) : consumer
Enter MASTER_IPADDR (ex xxx.xxx.xxx.xxx) : 128.199.242.74
```

Press enter to continue the process.

### What's Next

Head over to [Getting Started](../getting-started/index.md) page for an example on how to use Gluu Cluster API.


