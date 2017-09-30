---
title:  "Batch Train"
date:   2017-08-03 11:02:01 -0700
type: "spec"
command: "npm run train"
commandSub: "sscAPIBase, username, password, versionIdCSV, batchSize"
---
## Overview
Application versions can be sent for training to the Fortify Audit Assistant service. 
This script allows the sending of multiple batches of versions to the training service through SSC.

The script splits the versions into batches, each firing in parallel training requests on its own.
Once all responses for a batch are returned, the next batch is sent out.

A sample run would look something like this:

<pre><code class="javascript">
//config.batchSize = 3
//versionIdCSV = 5432, 2123, 10, 653, 232, 123
----10:00:00----
Batch 1
 train 5432 //versionid=5432
 train 2123
 train 10
//all responses come back
----10:01:00----
Batch 2
 train 653
 train 232
 train 123
</code></pre>

> This document will not cover the code that is in ```commonTestUtils.js``` to handle the batch sending work.

## Training Action
At a minimum, a version must include some basic naming info and the application name (application is referred to as "project" in legacy releases).
The application is a logical container for versions to be used for reporting / filtering / grouping, and so on (for example, Application=Office, version=95, version-2017)

<pre><code class="javascript">
    //restClient.js
    /**
     * send for prediction or training or future actions when implemented
     * @param {*} versionId 
     * @param {*} actionType 
     */
    sendToAA(versionId, actionType) {
        const restClient = this;
        return new Promise((resolve, reject) => {
            if (!restClient.api) {
                return reject("restClient not initialized! Make sure to call initialize before using API");
            }

            /* 
                call the action to train  - @type can be "sendForPrediction" or "sendForTraininig"
            */

            restClient.api["project-version-controller"].doActionProjectVersion({
                "id": versionId, resourceAction: { "type": actionType }
            }, getClientAuthTokenObj(restClient.token)).then((resp) => {
                if (resp.obj.data.message.indexOf("failed") !== -1) {
                    //legacy
                    reject(new Error(resp.obj.data.message));
                } else {
                    resolve(resp);
                }
            }).catch((error) => {
                reject(error);
            });;
        });
    }
</code></pre>
