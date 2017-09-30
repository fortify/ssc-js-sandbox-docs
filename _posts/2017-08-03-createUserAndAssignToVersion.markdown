---
title:  "Create Local User And Assign"
date:   2017-08-03 11:02:13 -0700
type: "spec"
command: "npm run createLocalUserAndAssignToVersion"
commandSub: "sscAPIBase, username, password, sampleVersionId"
---
## Overview
This test case will create a local SSC user and then assign to an existing application version.
1. Create the local user.
2. Assign to the version id defined in the config file.

## Creating the local user
A ```timestap``` is generated and appended to username, first and last name to differentiate the users being created between executions of the test.

<pre><code class="javascript">
//createLocalUserAndAssignToVersion_spec.js
    ...
    const localUser = {
        "requirePasswordChange": false,
        "userName": "newuser-"+timeStamp,
        "firstName": "new",
        "lastName": "user" +timeStamp,
        "email": "newuser" + +timeStamp + "@test.com",
        "clearPassword": "Admin_12",
        "passwordNeverExpire": true,
        "roles": [{
            "id": "securitylead"
        },
        {
            "id": "manager"
        },
        {
            "id": "developer"
        }]
    };
    ...
//restClient.js
    ...
    restClient.api["local-user-controller"].createLocalUser({
       user: localUser
    }, getClientAuthTokenObj(restClient.token)).then((resp) => {
    ...
</code></pre>

## Assigning the User
Similar to the ```Assign User to Versions``` Scenario.
the ```localUserEntity``` is saved from the previous test case.

<pre><code class="javascript">
//createLocalUserAndAssignToVersion_spec.js
    ...
    it('assign user to version', function (done) {
    const requestData = {"type":"assign","ids":[config.sampleVersionId]};
    
    if(!localUserEntity) {
      return done(new Error("No local user created."));
    }
    let userId = localUserEntity.id;
    restClient.assignUserToVersions(userId,requestData).then((resp) => {
      console.log(chalk.green(`successfully assigned user to version  ${config.sampleVersionId}`));
      done();
    ...
</code></pre>
