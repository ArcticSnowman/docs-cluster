## Cluster Management using Web Interface

The web interface provides a user friendly way of using the API and managing the various resources of the cluster.

### Installation

Install all required packages:

```sh
echo "deb http://repo.gluu.org/ubuntu/ trusty-devel main" > /etc/apt/sources.list.d/gluu-repo.list
curl http://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
apt-get update
apt-get install -y gluu-webui apache2 libapache2-mod-wsgi
```

As `gluu-webui` is configured to run behind Apache HTTPD and `mod_wsgi`, we need to create WSGI script
to load the application:

```sh
mkdir -p /var/www/gluuwebui
echo 'from gluuwebui import app as application' > /var/www/gluuwebui/gluuwebui.wsgi
```

Note, binding Apache HTTPD to port 80 is discouraged since it may conflict with port used by nodes;
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

### Accessing the Interface

As `gluuwebui` is running in localhost, one possible way to access the interface is by using SSH tunneling:

```sh
ssh -L 8800:localhost:8800 root@gluuwebui.example.com
```

After the connection to gluuwebui.example.com:8800 is established, visit `http://localhost:8800` in web browser.

### Using the Interface

The interface provides access to managing the resources of the cluster with a list of items on the left side. Clicking on a reource type takes you to the overview page of that particular resource.

#### The overview page

The overview page consists of a table which lists all the entities of that particular resource type, with their most important details and their associated actions if any. If the API provides only POST and DELETE actions then the interface provides a 'New RESOURCE' button on top and a 'DELETE' button for each resource in the table.


![Overview with only Delete](../../img/webui_overview1.png)

If the API also provides a PUT action then a EDIT button is also show in the actions column, which can be used to edit the information of that particular resource.

![Overview with Edit and Delete](../../img/webui_overview2.png)

The overview table allows you to see the complete details by clicking on its name.

![Detail view](../../img/webui_cluster_details.png)

#### Adding/Editing Resources

New resources can be added by clicking the 'New RESOURCE' buttons in the overview pages. This takes you to the form where all the values required for that particular resource can put entered. Submitting the form requests the API for action. If the resource creation is successful, you are taken back to the overview page and you can see the new resource listed in the overview table.

![New Resource form](../../img/webui_new_form.png)

Existing resources can be altered when they provide an Edit action. Clicking the Edit button takes you to the edit form which is similar to the 'New' form with the ID of the existing resource disabled. ID's are unique system generated strings and hence disabled from editing.

![Edit form](../../img/webui_edit_form.png)
