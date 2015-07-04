## Cluster Management using Web Interface

The web interface provides a user friendly way of using the API and managing the various resources of the cluster.

### Installation

### Accessing the Interface

### Using the Interface

The interface provides access to managing the resources of the cluster with a list of items on the left side. Clicking on a reource type takes you to the overview page of that particular resource. 

#### The overview page

The overview page consists of a table which lists all the entities of that particular resource type, with their most important details and their associated actions if any. If the API provides only POST and DELETE actions then the interface provides a 'New RESOURCE' button on top and a 'DELETE' button for each resource in the table.


![Overview with only Delete](/img/webui_overview1.png)

If the API also provides a PUT action then a EDIT button is also show in the actions column, which can be used to edit the information of that particular resource.

![Overview with Edit and Delete](/img/webui_overview2.png)

The overview table allows you to see the complete details by clicking on its name.

![Detail view](/img/webui_cluster_details.png)

#### Adding/Editing Resources

New resources can be added by clicking the 'New RESOURCE' buttons in the overview pages. This takes you to the form where all the values required for that particular resource can put entered. Submitting the form requests the API for action. If the resource creation is successful, you are taken back to the overview page and you can see the new resource listed in the overview table.

![New Resource form](/img/webui_new_form.png)

Existing resources can be altered when they provide an Edit action. Clicking the Edit button takes you to the edit form which is similar to the 'New' form with the ID of the existing resource disabled. ID's are unique system generated strings and hence disabled from editing.

![Edit form](/img/webui_edit_form.png)
