# Camunda Sharepoint / OneDrive Custom Connector Template
This project contains a custom Camunda 8 Connector template that provides access to SharePoint or OneDrive operations. The operations and functionality have to be encoded into the template to provide the integration.

This readme outlines how to update the template to provide the operation functionality required.

# How does the connector work?
The Connector is based on the `io.camunda:http-json` connector used by various OOTB connectors such as the REST API Connector. Integration with Sharepoint and OneDrive within this Connector template is achieved using the Microsoft Graph API. Details of the API can be found in the Graph API reference here: https://learn.microsoft.com/en-us/graph/api/overview?view=graph-rest-1.0

The Connector template exposes some of the standard REST properties such as those to define the OAUTH2 authentication credentials, as well as mapping the response and error messages that come back from the service call. It also provides specific properties that are dynamically generated based on the operation selected by the user using the Connector, for example, the Graph API end point URL.

# How to add a new operation to the connector template?
The following steps outline how to add a new operation to the Connector.

## Adding the operation option
Find the property with the id `sharePointOperation`, and add your new operation to the list of choices, for example `createFolder`:

```json
{
  "label": "SharePoint Operation",
  "id": "sharePointOperation",
  "description": "The operation in SharePoint to invoke",
  "group": "sharePointOperation",
  "type": "Dropdown",
  "choices": [
    {
      "name": "Create Folder",
      "value": "createFolder"
    },
    {
      "name": "Get Root Site",
      "value": "getRootSite"
    }
  ],
  "binding": {
    "type": "zeebe:input",
    "name": "sharePointOperation"
  }
}
```

## Adding operation to HTTP method
Depending on the HTTP method the operation uses, add it to the `condition.oneOf` properties, for example:

```json
{
  "description": "This sets the HTTP method to POST based on the SP operation that is set",
  "id": "postMethod",
  "type": "Hidden",
  "value": "post",
  "condition": {
    "oneOf": [
      "createFolder"
    ],
    "property": "sharePointOperation"
  },
  "binding": {
    "type": "zeebe:input",
    "name": "method"
  }
}
```

## Add custom properties to operation group
Next add any custom properties required for your operation. The crucial thing here is to make sure that the following are set:
- `group`: This ordinarily should be set to `sharePointOperation` so they are all grouped together in the same place in the properties panel.
- `condition`: This is configured to only show the property editor either specific operations (but could be used by more than one).
- `binding.name`: The name used will be referenced elsewhere in the Connector config such as in the URL or request payload.
- `constraints.notEmpty`: Is this property mandatory or not?

```json
{
  "description": "This is the Id of the Sharepoint / OneDrive site.",
  "label": "Site Id",
  "group": "sharePointOperation",
  "type": "String",
  "feel": "optional",
  "binding": {
    "type": "zeebe:input",
    "name": "siteId"
  },
  "condition": {
    "type": "simple",
    "property": "sharePointOperation",
    "equals": "createFolder"
  },
  "constraints": {
    "notEmpty": true
  }
}
```

## Add end point URL generation
Each operation will require an end point URL, which might be static, or dynamic based on the paramater values passed in. The example below shows the parameters plugged into the URL (this is done at runtime and not at design time). Note how the start of the `value` property has `=\"` (and ends with a `"`), which allows the URL to be dynamically created by concatenating variables and strings as an expression.

Some operations may have a fixed URL, in which case that can just be enclosed inside outer quotes like any other string JSON property.

Finally, make sure that the correct operation name is included in the `condition` so only this URL is passed to the connector when the operation is selected.

```json
{
  "id": "createFolderURL",
  "type": "Hidden",
  "value": "=\"https://graph.microsoft.com/v1.0/sites/\"+siteId+\"/drive/items/\"+parentItemId+\"/children/\"",
  "binding": {
    "type": "zeebe:input",
    "name": "url"
  },
  "condition": {
    "property": "sharePointOperation",
    "equals": "createFolder"
  }
}
```

## Adding default request payloads
If the operation is a POST then it will have a request payload (in JSON). These can be defaulted to contain default values or structure that can be changed to suit, as shown in the example below.

Make sure that the correct operation name is included in the `condition` so only this URL is passed to the connector when the operation is selected.

```json
{
  "label": "Request body",
  "description": "Payload to send with the request",
  "group": "sharePointOperation",
  "type": "Text",
  "feel": "optional",
  "value": "={\n\t\"name\":\"My Folder\",\n\t\"folder\": {},\n\t\"@microsoft.graph.conflictBehavior\": \"rename\"\n}",
  "binding": {
    "type": "zeebe:input",
    "name": "body"
  },
  "condition": {
    "property": "sharePointOperation",
    "oneOf": [
      "createFolder"
    ]
  }
}
```

# Fork the repository
To contribute to the project you should fork the repository and raise a pull request.

## Raise a pull request
In the description of your pull request please supply the following details, with screen shots where possible:

* Provide a desription of the change you have made
* Submit results of testing you have done as part of the change
