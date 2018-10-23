---
title:  "Update Custom Tags of Version"
date:   2017-08-03 11:02:13 -0700
type: "spec"
command: "npm run updateCustomTagsOfVersion"
commandSub: "sscAPIBase, username, password, sampleVersionId, sampleCustomTagId"
---
## Overview
This test case will get all custom tags of an existing application version, will get current object of selected custom tag and will append the custom tag to existing custom tag list of the application version.
1. Get all Custom Tags of existing application version.
2. Get an existing Custom Tag by ID and append it to Custom Tag list.
3. Update the Custom Tags list of application version.
## Get all Custom Tags of existing application version
We obtain a list with all current Custom Tags related to a Project Version.
The variable *customTags* will store the related Custom Tags.
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
## Get existing Custom Tag by ID and append to Custom Tag list
We need to get the existing Custom Tag Object with the ID that we've set up.
Once we have the complete object, We can append it to the project version Custom Tags list.
The variable *customTags* will carry Custom Tags List with the new Custom Tag Object appended.
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
## Update the Custom Tags list of application version
In this section we will finally update the Custom Tags list related to Project Version.
You may notice that any duplicate Custom Tag in list is ignored by API.
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
