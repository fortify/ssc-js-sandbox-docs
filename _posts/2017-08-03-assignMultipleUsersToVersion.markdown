---
title:  "Assign Multiple Users"
date:   2017-08-03 11:02:13 -0700
type: "spec"
command: "npm run assignAppVersionToAuthEntities"
commandSub: "sscAPIBase, username, password, sampleAuthEntities, sampleSecondaryVersionId"
---
## Overview

> Auth entities are a representation of a local user, LDAP user or LDAP group.

This test cases will get auth entities (see below) for a version, assign new entities to the same version and then revert back to original state.
1. Get the current assigned auth entities for version ```config.sampleSecondaryVersionId``` and save result.
2. Assign new auth entities from ```config.sampleAuthEntities``` to ```config.sampleSecondaryVersionId```.
3. Revert back to original state of version by re-assigning the same entities saved from the response in step 1.

> A ```PUT``` (update) operation on version auth entities replaces the whole list of assigned auth entities, no merges are done.

## Get Version Assigned Auth Entities
Get the list of auth entities and save to ```originalAuthEntities```

<pre><code class="javascript">
//restClient.js
    ...
    restClient.api["auth-entity-of-project-version-controller"].listAuthEntityOfProjectVersion({parentId: pvId}, 
        getClientAuthTokenObj(restClient.token))
    ...      
 //assignAppVersionToAuthEntities_spec.js         
    ...
        originalAuthEntities = resp
    ...
</code></pre>

## Assign New Auth Enitities
Pass a list of auth entities (represents a local user / LDAP user or LDAP group) to the API call.

<pre><code class="javascript">
//config.js
    ...
    sampleAuthEntities: [{"id": 12, "isLdap": false}, {"id": 9, "isLdap": false}, {"id": 6, "isLdap": false}]
    ...
//restClient.js    
    ...
    restClient.api["auth-entity-of-project-version-controller"].updateCollectionAuthEntityOfProjectVersion({
                parentId: pvId,
                data: arrayOfAuthEntities //== sampleAuthEntities
            }, getClientAuthTokenObj(restClient.token)
            ).then((resp) => {
    ...
</code></pre>


## Revert
Re-assign the auth entities saved in the previous test in ```originalAuthEntities``` to the same version.

<pre><code class="javascript">
//restClient.js    
    ...
    restClient.api["auth-entity-of-project-version-controller"].updateCollectionAuthEntityOfProjectVersion({
                parentId: pvId,
                data: arrayOfAuthEntities //== originalAuthEntities
            }, getClientAuthTokenObj(restClient.token)
            ).then((resp) => {
    ...
</code></pre>
