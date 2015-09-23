[TOC]
# Cluster Management using Web Interface

The web interface provides a user friendly way of using the API and managing the various resources of the cluster.

## Installation
The installation of the web interface is covered in the Installation Section.

* [Install Web Interface Package](http://www.gluu.org/docs-cluster/admin-guide/installation/#gluu-cluster-web-interface)

## Accessing the Interface
To log into the web interface, it is necessary to ssh into the master provider, as the interface is run locally and it is not facing the internet for security reasons.

Run the following command to SSH for accessing the web interface:

`ssh -L <port-name>:localhost:8800 -i <cluster-name> ubuntu@<cluster-domain>`

Point your browser to the following address to access the webui:

`http://localhost:<port-name`

## Using the Web Interface
### The overview page

The overview page consists of a table which lists all the entities of that particular resource type, with their most important details and their associated actions if any. If the API provides only POST and DELETE actions then the interface provides a 'New RESOURCE' button on top and a 'DELETE' button for each resource in the table.


![Overview with only Delete](../../img/webui_overview1.png)

If the API also provides a PUT action then a EDIT button is also show in the actions column, which can be used to edit the information of that particular resource.

![Overview with Edit and Delete](../../img/webui_overview2.png)

The overview table allows you to see the complete details by clicking on its name.

![Detail view](../../img/webui_cluster_details.png)

### Adding/Editing Resources

New resources can be added by clicking the 'New RESOURCE' buttons in the overview pages. This takes you to the form where all the values required for that particular resource can put entered. Submitting the form requests the API for action. If the resource creation is successful, you are taken back to the overview page and you can see the new resource listed in the overview table.

![New Resource form](../../img/webui_new_form.png)

Existing resources can be altered when they provide an Edit action. Clicking the Edit button takes you to the edit form which is similar to the 'New' form with the ID of the existing resource disabled. ID's are unique system generated strings and hence disabled from editing.

![Edit form](../../img/webui_edit_form.png)
