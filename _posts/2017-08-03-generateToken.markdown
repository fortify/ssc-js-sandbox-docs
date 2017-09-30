---
title:  "Generate a token"
date:   2017-08-03 11:01:17 -0700
type: "spec"
command: "npm run generateToken"
commandSub: "sscAPIBase, username, password, FORTIFY_TOKEN_TYPE (env var)"
---
## Overview
Generate an authentication token based on type.
Default is to generate a UnifiedLoginToken.

You can set the FORTIFY_TOKEN_TYPE environment variable before running the script to generate the different token types, such as:
AnalysisDownloadToken, AnalysisUploadToken, AuditToken, UploadFileTransferToken, DownloadFileTransferToken, ReportFileTransferToken, CloudCtrlToken, CloudOneTimeJobToken, WIESystemToken, WIEUserToken, UnifiedLoginToken, ReportToken, PurgeProjectVersionToken

### Getting a file upload token
Using the Auth controller to generate the token.
Use basic http auth to generate token

<pre><code class="javascript">
    generateToken(type) {
        const restClient = this;
        return new Promise((resolve, reject) => {
            const auth = 'Basic ' + new Buffer(config.user + ':' + config.password).toString('base64');

            restClient.api["auth-token-controller"].createAuthToken({ authToken: { "terminalDate": getExpirationDateString(), "type": type } }, {
                responseContentType: 'application/json',
                clientAuthorizations: {
                    "Basic": new Swagger.PasswordAuthorization(config.user, config.password)
                }
            }).then((data) => {
                //got it so pass along
                resolve(data.obj.data.token)
            }).catch((error) => {
                reject(error);
            });
        });
    }
</code></pre>


