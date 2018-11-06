---
title:  "Update Custom Tags of Version"
date:   2017-08-03 11:02:13 -0700
type: "spec"
command: "npm run updateCustomTagsOfVersion"
commandSub: "sscAPIBase, username, password, sampleVersionId, sampleCustomTagId"
---
## Overview
The test case described in this article does the following:
1. Retrieves all custom tags for the application version
2. Retrieves a custom tag based on its ID and appends it to custom tag list
3. Updates the custom tags list for the application version

The following sections describe how to perform these steps.

## Obtaining all Custom Tags for an Application Version

To obtain a list of all custom tags associated with an application version, run the following:

<pre><code class="javascript">
//updateCustomTagsOfVersion_spec.js
    ...
    let customTags = [];
    it('get all the custom tags for a project', function (done) {
      restClient.getAllCustomTagsOfVersion([config.sampleVersionId]).then((response) => {
        customTags = response;
        console.log(chalk.green(`successfully got all the custom tags for the project version with ID ${config.sampleVersionId}`));
        done();
      }).catch((err) => {
        console.log(chalk.red("error getting all the custom tags for the project version with ID:" + config.sampleVersionId), err)
        done(err);
      });
    });
    ...
//restClient.js
    ...
    getAllCustomTagsOfVersion(versionId) {
        const controller = this.api["custom-tag-of-project-version-controller"];
        return controller.listCustomTagOfProjectVersion({ 'parentId': versionId }, getClientAuthTokenObj(this.token)).then((response) => {
            return response.obj.data;
        });
    }
    ...
</code></pre>

The variable customTags stores the custom tags.

## Obtaining a Custom Tag and Adding it to the Custom Tag List

Next, you obtain the custom tag object based on its the ID. Once you have the complete object, you can append it to the application version’s custom tags list. The variable customTags stores the custom tags list with the new custom tag object appended.

To obtain the custom tag object based on its ID:

<pre><code class="javascript">
//updateCustomTagsOfVersion_spec.js
    ...
    it('get the specific custom tag that we want to add and append to final list', function (done) {
      restClient.getCustomTag([config.sampleCustomTagId]).then((response) => {
        customTags.push(response.obj.data);
        console.log(chalk.green(`IDs of custom tag list with new appended element: ` + customTags.map(elem => elem.id).join(", ")));
        done();
      }).catch((err) => {
        console.log(chalk.red("error appending the selected custom tag to the list"), err)
        done(err);
      });
    });
    ...
//restClient.js
    ...
    getCustomTag(customTagId) {
        const controller = this.api["custom-tag-controller"];
        return controller.readCustomTag({ 'id': customTagId }, getClientAuthTokenObj(this.token)).then((response) => {
            return response;
        });
    }
    ...
</code></pre>

## Updating the Custom Tags List for an Application Version

To update the custom tags list for the application version:

<pre><code class="javascript">
//updateCustomTagsOfVersion_spec.js
    ...
    it('update the list of custom tags', function (done) {
      restClient.updateCustomTagsOfVersion([config.sampleVersionId], customTags).then((response) => {
        console.log(chalk.green(`successfully got all the custom tags for the project version with ID ${config.sampleVersionId}`));
        done();
      }).catch((err) => {
        console.log(chalk.red("error updating the list of custom tags for the project version with ID:" + config.sampleVersionId), err)
        done(err);
      });
    });
    ...
//restClient.js
    ...
    updateCustomTagsOfVersion(versionId, customTagList) {
      const controller = this.api["custom-tag-of-project-version-controller"];
      return controller.updateCollectionCustomTagOfProjectVersion({ 'parentId': versionId, 'data': customTagList}, getClientAuthTokenObj(this.token)).then((response) => {
          return response;
      });
    }
    ...
</code></pre>

Note that the API ignores any duplicate custom tags in the list.
