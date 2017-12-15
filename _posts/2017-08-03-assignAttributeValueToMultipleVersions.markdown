---
title:  "Batch Attribute"
date:   2017-08-03 11:02:12 -0700
type: "spec"
command: "npm run assignAttributeVersions_spec"
commandSub: "sscAPIBase, username, password, versionIdCSV, batchSize, sampleAttributeValue"
---

## Overview
> Attribute are created in the SSC administration section under Templates -> Attributes. 

The script:
1. creates a new attribute definition of type single input (text) 
2. splits the versions into batches, each firing an attribute value assignment request in parallel.
Once responses are returned for all versions in a batch, the next batch is sent out.

> To learn more about the batch functionality refer to the ```Batch Predict``` section.

## Setup

This script creates an attribute with a random name and then saves it's ```Id``` for assignment part.

An easy way to get the list of existing attribute defenitions is to run an HTTP GET API call to ```[host:port]/[ssc context]/api/v1/attributeDefinitions```.
Sniffing the http trafic in the UI is also possible to get the attribute id.

The Id from the create step of the script will be used in the assignment part wuth the ```sampleAttributeValue```. For example:
```sampleAttributeValue: [{"attributeDefinitionId":29,"values":null,"value":"#sandbox_sample_guid"}]```

The following code will run through the csv file (defined int he config.js file) and for each version will assign the attribute value ```my special value``` to attribute ```29```.

<pre><code class="javascript">
    //restClient.js 

    /**
     * create attribute definition
     * @param {*} definition - attribute definition json payload (see config.js)
     */
    createAttributeDefinition(definition) {
        const restClient = this;
        return new Promise((resolve, reject) => {
            if (!restClient.api) {
                return reject("restClient not initialized! make sure to call initialize before using API");
            }
            restClient.api["attribute-definition-controller"].createAttributeDefinition({
                resource: definition
            }, getClientAuthTokenObj(restClient.token)).then((resp) => {
                console.log("attribute " + resp.id + " created successfully!");
                resolve(resp.obj.data); //return attribute definition object
            }).catch((err) => {
                reject(err);
            });
        });
    }

     /**
     * Assign an attribute to a version with value
     * @param {*} versionId 
     * @param {*} attributes 
     */
    assignAttribute(versionId, attributes) {
        const restClient = this;
        return new Promise((resolve, reject) => {
            if (!restClient.api) {
                return reject("restClient not initialized! make sure to call initialize before using API");
            }
            restClient.api["attribute-of-project-version-controller"].updateCollectionAttributeOfProjectVersion({
                parentId: versionId,
                data: attributes
            }, getClientAuthTokenObj(restClient.token)).then((resp) => {
                console.log("versionid " + versionId + " updated successfully!");
                resolve(versionId);
            }).catch((err) => {
                reject(err);
            });
        });
    }

    //assignAttributeVersions_spec.js - second part of script to assign values.
    it('batch assigns a value to an attribute on multiple versions', function (done) {
        commonTestsUtils.batchAPIActions(done, "assignAttribute", config.sampleAttributeValue);
    });
</code></pre>
