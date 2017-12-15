---
title:  "Assign an Attribute Value to Multiple Versions"
date:   2017-08-03 11:02:12 -0700
type: "spec"
command: "npm run assignAttributeVersions_spec"
commandSub: "sscAPIBase, username, password, versionIdCSV, batchSize, sampleAttributeValue"
---

## Overview
> Attribute are created in the SSC administration section under Templates -> Attributes. 

The script splits the versions into batches, each firing an attribute value assignment request in parallel.
Once responses are returned for all versions in a batch, the next batch is sent out.

> To learn more about the batch functionality refer to the ```Batch Predict``` section.

## Setup

This script assumes that an attribute exists and that you know it's id.
An easy way to get this information is to run an HTTP GET API call to ```[host:port]/[ssc context]/api/v1/attributeDefinitions```.
Sniffing the http trafic in the UI is also possible to get the attribute id.

Once you have the Id, you can set it in the config.js file under ```sampleAttributeValue```. For example:
```sampleAttributeValue: [{"attributeDefinitionId":29,"values":null,"value":"my special value"}]```

The following code will run through the csv file (defined int he config.js file) and for each version will assign the attribute value ```my special value``` to attribute ```29```.

<pre><code class="javascript">
    //restClient.js
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


    //assignAttributeVersions_spec.js
    it('batch assigns a value to an attribute on multiple versions', function (done) {
        commonTestsUtils.batchAPIActions(done, "assignAttribute", config.sampleAttributeValue);
    });
</code></pre>
