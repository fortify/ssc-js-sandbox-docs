---
title:  "Assign User to Versions"
date:   2017-08-03 11:02:15 -0700
type: "spec"
command: "npm run assignUserToVersions"
commandSub: "sscAPIBase, username, password, sampleVersionId, sampleSecondaryVersionId"
---
## Overview
Assign a user based on user id to multiple versions in a single API call.

- Userid is fetched from configuration object.
- Action doActionProjectVersionOfAuthEntity is passed ```type="assign"``` action with array of version ids.

<pre><code class="javascript">
////assignUserToVersions_spec.js
    const userId = config.sampleUserId;
    const requestData = {"type":"assign","ids":[config.sampleVersionId, config.sampleSecondaryVersionId]};
    restClient.assignUserToVersions(userId,requestData).then((resp) => {
      console.log(chalk.green(`successfully assigned userid ${userId} to versions [${config.sampleVersionId} , ${config.sampleSecondaryVersionId}]`));
    ...
//restClient.js
    restClient.api["project-version-of-auth-entity-controller"].doActionProjectVersionOfAuthEntity({
                parentId: userId,
                collectionAction: requestData
    ...

</code></pre>