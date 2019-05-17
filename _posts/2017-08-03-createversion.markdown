---
title:  "Create Version"
date:   2017-08-03 11:03:14 -0700
type: "spec"
command: "npm run createVersion"
commandSub: "sscAPIBase, username, password, sampleVersionId (copy state)"
---
## Overview
Creating a version in SSC is a three-part process.
1. create the bare bones resource
2. assign attributes / users / templates
3. commit the version (ready for use)

> One optional fourth step can be added if the ```options.copyCurrentState``` is set to ```true```, to copy applicaiton state from an existing version.

> ``async.waterfall`` is not covered here. If you want to know more about it, see the Authentication section, which explains it better.

> The following code blocks show the *bare minimum* required to create a working application version in SSC. There are many assignable parameters, such as bug tracker integrations, custom tags, and so on.

## Creating the version resource
At a minimum, a version must include some basic naming info and the application name (application is referred to as "project" in legacy releases). The application is a logical container for versions to be used for reporting / filtering / grouping, and so on (for example, Application=Office, version=95, version-2017)

<pre><code class="javascript">
//restClient.js
    restClient.api["project-version-controller"].createProjectVersion({
        resource: {
            "name": options.name,
            "description": options.description,
            "active": true,
            "committed": false,
            "project": {
                "name": options.appName,
                "description": options.appDesc,
                "issueTemplateId": options.issueTemplateId
            },
            "issueTemplateId": options.issueTemplateId
        }
</code></pre>

## Assigning attributes
Each version has a default set of attributes assigned to it. Some of these are *mandatory* to commit the version to SSC.
> To find out which attributes your SSC instance has set up, log in to SSC, go to the Administraton view, and then look under Templates.

In this case, we pass to the data parameter the options.attributes.
<pre><code class="javascript">
//restClient.js
     function assignAttributes(version, callback) {
            restClient.api["attribute-of-project-version-controller"].updateCollectionAttributeOfProjectVersion({
                parentId: version.id,
                data: options.attributes
            }, getClientAuthTokenObj(restClient.token)).then((resp) => {
                callback(null, version, resp.obj.data);
            }).catch((err) => {
                callback(err);
            });
        },
</code></pre>
If you look at ```createVersion_spec.js``` you will see that the following is passed:

<pre><code class="javascript">
//createVersion_spec.js
   {
      ...
      attributes: [
        { attributeDefinitionId: 5, values: [{ guid: "New" }], value: null },
        { attributeDefinitionId: 6, values: [{ guid: "Internal" }], value: null },
        { attributeDefinitionId: 7, values: [{ guid: "internalnetwork" }], value: null },
        { attributeDefinitionId: 1, values: [{ guid: "High" }], value: null }
      ]
    }
</code></pre>


## Committing the version
Once the required attributes are set on the newly-created resource, you can commit the version.
> Uncommitted versions are not usable in the UI. For example, artifacts cannot be uploaded.

<pre><code class="javascript">
//restClient.js
    function commit(version, attrs, callback) {
        restClient.api["project-version-controller"].updateProjectVersion({
            id: version.id,
            resource: { "committed": true }
        }, getClientAuthTokenObj(restClient.token)).then((resp) => {
            callback(null, resp.obj.data);
        }).catch((err) => {
            callback(err);
        });
    }
</code></pre>



## Copy current state (optional)
if the ```options.copyCurrentState``` is set to true, one additional call will be added to the waterfall queue.
This call tells the server to copy apllication state (issues) from an existing version (in this case config.sampleVersionId).
Notice the ```previousProjectVersionId: config.sampleVersionId``` which is passed to the action command.

<pre><code class="javascript">
//restClient.js
   //if copyCurrentState is passed then add a funnction to copy state
    //this will activate a background process on the server to copy over issues.
    if (options.copyCurrentState === true) {
        waterfallFunctionArray.push(function (version, callback) {
            restClient.api["project-version-controller"].doActionProjectVersion({
                "id": version.id, resourceAction: {
                    "type": "COPY_CURRENT_STATE",
                    values: { projectVersionId: version.id, previousProjectVersionId: config.sampleVersionId, copyCurrentStateFpr: true }
                }
            }, getClientAuthTokenObj(restClient.token)).then((resp) => {
                if (resp.obj.data.message.indexOf("failed") !== -1) {
                    callback(new Error(resp.obj.data.message))
                } else {
                    callback(null, version);
                }
            }).catch((error) => {
                reject(error);
            });;
        });
    }
</code></pre>
