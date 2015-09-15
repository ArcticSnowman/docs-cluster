# Installation
[TOC]

The Gluu Cluster is suported for Ubuntu for now. There are three packages that completes the cluster; the master, the consumer and the cluster API. The master package and the cluster API must be installed in the same host. The consumer package is not necessary for a single server installation. There is one additional package, gluu-webui, that provides a user friendly way of using the API and managing the cluster. 

## Gluu Cluster Master

There are various components that make the cluster possible. The components are listed in the [components](http://www.gluu.org/docs-cluster/admin-guide/components/) page. The components are packaged into a single `deb` package to minimize the complexity of the situation. The package is named `gluu-master`.

### Installing gluu-master

Run the following commands to install the gluu-master package:
```
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list
curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
apt-get install -y gluu-master
```
### Configuring gluu-master

A script called the `postinstall.py` should be downloaded and run to configure the gluu-master package. 

Run the following command to download the script:

`wget https://raw.githubusercontent.com/GluuFederation/gluu-cluster-postinstall/master/postinstall.py`

Make a note of the IP address of the master package as it will be used in the script. Run the script using the following command:

`python postinstall.py`

3. Enter the values asked by the prompt when the script is run, and press enter to continue. An example of the script's pormpt is given below:

```
root@gluu-master:~# python postinstall.py
Enter host type (ex. master or consumer) : master
Enter MASTER_IPADDR (ex xxx.xxx.xxx.xxx) : 128.199.242.74
IP address of this server: 128.199.242.74
Password for TLS certificate:
Re-type password for TLS certificate:
```

## Installing Cluster API

The cluster API, `gluu-flask` must be installed in the same host as gluu-master.

Run the following command to install gluu-flask:
`apt-get install -y gluu-flask`

The command will install a daemon which is automatically started by init script. There are a few commands that are available for gluu-flask. The available commands are

	service gluu-flask start
	service gluu-flask restart
	service gluu-flask stop
	service gluu-flask status

## Gluu Cluster Consumer

The consumer package consists of various componenents listed in the [components page](http://www.gluu.org/docs-cluster/admin-guide/components/). The components are packaged into a single `deb` package called gluu-consumer.

### Installing gluu-consumer
Run the following commands to install gluu-consumer:
```
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list
curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
apt-get install -y gluu-consumer
```
### Configuring gluu-consumer
A script called the `postinstall.py` should be downloaded and run to configure the gluu-consumer package.

Run the following command to download the script:

`wget https://raw.githubusercontent.com/GluuFederation/gluu-cluster-postinstall/master/postinstall.py`

Make a note of the IP address of the gluu-master before running the script. Run the script using the following command:
	
`python postinstall.py`

Enter the values asked by the prompt when the script is run, and press enter to continue. An example of the script's prompt is given below:
```
root@gluu-master:~# python postinstall.py
Enter host type (ex. master or consumer) : consumer
Enter MASTER_IPADDR (ex xxx.xxx.xxx.xxx) : 128.199.242.74
IP address of this server: 128.199.242.75
Password for TLS certificate:
Re-type password for TLS certificate:
```
## Gluu Cluster Web Interface

The Cluster Web Interface package, gluu-webui, offers a user interface accessible via web to manage the cluster using the cluster API.

### Installing gluu-webui

Run the following commands to install gluu-webui:
```
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list
curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
apt-get install -y gluu-webui apache2 libapache2-mod-wsgi
```

### Configuring gluu-webui
As `gluu-webui` is configured to run behind Apache HTTPD and `mod_wsgi`, a WSGI script must be created
to load the application:

```sh
mkdir -p /var/www/gluuwebui
echo 'from gluuwebui import app as application' > /var/www/gluuwebui/gluuwebui.wsgi
```

Note: binding Apache HTTPD to port 80 is discouraged since it may conflict with port used by nodes;
hence we need to change the default port for Apache HTTPD. Modify the contents of `/etc/apache2/ports.conf` as seen below:

```apache
# /etc/apache2/ports.conf
Listen 127.0.0.1:8800

<IfModule ssl_module>
    Listen 127.0.0.1:8443
</IfModule>

<IfModule mod_gnutls.c>
    Listen 127.0.0.1:8443
</IfModule>
```

Afterwards, create a virtualhost and save to `/etc/apache2/sites-available/gluuwebui_apache.conf`:

```apache 
# /etc/apache2/sites-available/gluuwebui_apache.conf
<VirtualHost 127.0.0.1:8800>
    ServerAdmin admin@example.com
    ServerName gluuwebui.example.com

    WSGIDaemonProcess gluuwebui threads=5 display-name=%{GROUP}
    WSGIProcessGroup gluuwebui

    WSGIScriptAlias / /var/www/gluuwebui/gluuwebui.wsgi

    <Directory /var/www/gluu-webui>
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```

Note, the virtualhost above running in 127.0.0.1 (localhost) as currently there's no protection mechanism for `gluuwebui` application yet.

After all settings have been configured, we need to restart the Apache HTTPD service:

```sh
# disable default Apache HTTPD virtualhost
a2dissite 000-default

# enable gluuwebui
a2ensite gluuwebui_apache

# restart the service
service apache2 restart
```

