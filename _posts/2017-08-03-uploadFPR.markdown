---
title:  "Upload FPR & Track Processing"
date:   2017-08-03 11:01:15 -0700
type: "spec"
command: "npm run uploadFPR"
commandSub: "sscAPIBase, username, password, sscUploadURL, sampleFPR, sampleFPRVersionId"
---
## Overview
Uploading an FPR is *not currently part of* the SSC Swagger spec and, therefore, is not generated in the code API layer.

This tutorial is based on the static documentation you can find in your SSC instance when navigating to 
[SSC URL]/<app context>/html/docs/docs.html#/fileupdownload

### Getting a file upload token
File uploads in SSC require a single use auth token. The steps to upload an FPR are:
1. Generate a file token.
2. Pass the token to the upload endpoint in a query param along with a multipart form upload request.

There are two ways to get the token. You can use the method using the ```fileTokens``` endpoint (Legacy) mentioned in the static docs, or use Swagger.
We will use Swagger.
> see [generateToken](#/2017/08/03/generateToken)
<pre><code class="javascript">
    /**
    * get the single upload token
    */
    function getFileToken(callback) {
        restClient.generateToken("UploadFileTransferToken").then(() => {
            callback(null, token);
        }).catch(() => {
            callback(error);
        });
    },
</code></pre>

### Uploading the file
As mentioned previously, this is not an API that is part of the Swagger spec at the moment.

> This sample uses the [node request](https://github.com/request/request) library to perform the multi part POST request.

<pre><code class="javascript">
//restClient.js
     function uploadFile(token, callback) {
        //Upload the file. For more information on this in SSC, navigate to [SSC URL]/%lt;app context&gt;/html/docs/docs.html#/fileupdownload
        const url = config.sscFprUploadURL + "?mat=" + token + "&amp;entityId=" + versionId;
        try {
            const formData = {
                files: [
                    fs.createReadStream(__dirname + "/../" + fprPath)
                ]
            };
            request.post({ url: url, formData: formData }, function optionalCallback(err, httpResponse, body) {
                if (err) {
                    callback(err);
                } else {
                    if (body &amp;&amp; body.toLowerCase().indexOf(':code&gt;-10001') !== -1) {
                        callback(null, body);
                    } else {
                        callback(new Error("error uploading FPR: " + body))
                    }

                }
            });
        } catch (e) {
            callback(e);
        }
    }
</code></pre>

### Tracking the status
To track the processing status of the artifact in SSC we will query the ```job-controller``` endpoint. 
For this example we will just print messages in the different processing phases. 

<pre><code class="javascript">
//fprUpload_spec.js
   /**
   * Poll the uploaded FPR in previous test for processing status
   */
  it('poll for status ', function (done) {
    
    if (!artifactJobid) {
      return done(new Error("previous upload FPR failed"));
    }
    /*
    * callback for setTimeout. Will use restClient to get the fpr status and then based on returned status either set another timer or stop execution.
    */
    function checkStatus() {
      const timeoutSec = 2;
      restClient.getJob(artifactJobid).then((job) => {
        jobEntity = job; //update for next test
        let msg;
        switch (jobEntity.state) {
          case "FAILED":
            msg = console.log(`${artifactJobid} failed! project version name: project version id: ${jobEntity.projectVersionId} artifact id: ${jobEntity.jobData.PARAM_ARTIFACT_ID}`);
            console.log(chalk.red(msg));
            setTimeout(checkStatus, timeoutSec * 1000);
            break;
          case "CANCELLED":
            msg = console.log(`${artifactJobid} canceled! project version id: ${jobEntity.projectVersionId} artifact id: ${jobEntity.jobData.PARAM_ARTIFACT_ID}`);
            console.log(chalk.red(msg));
            done(new Error(msg));
            break;
          case "RUNNING":
          case "WAITING_FOR_WORKER":
          case "PREPARED":
            console.log(`${artifactJobid} ongoing!`);
            setTimeout(checkStatus, timeoutSec * 1000);
            break;
          case "FINISHED":
            console.log(chalk.green(`${artifactJobid} completed successfully, project version id: ${jobEntity.projectVersionId} artifact id: ${jobEntity.jobData.PARAM_ARTIFACT_ID}`));
            done();
            break;
        }
      }).catch((err) => {
        let msg = `${artifactJobid} ongoing!`;
        console.log(chalk.red(msg));
        done(err);
      });
    }
    checkStatus();
  });
});
</code></pre>
