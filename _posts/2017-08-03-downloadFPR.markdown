---
title:  "Download FPR"
date:   2017-08-03 11:01:16 -0700
type: "spec"
command: "npm run uploadFPR"
commandSub: "sscAPIBase, username, password, sscUploadURL, sampleFPR, sampleFPRVersionId"
---
## Overview
Downloading an FPR file is *not currently part of* the SSC Swagger spec and, therefore, is not generated in the code API layer.

This tutorial is based on the static documentation you can find in your SSC instance when navigating to 
[SSC URL]/<app context>/html/docs/docs.html#/filedownload

### Getting a file download token
File downloads in SSC require a single use auth token. The steps to download an FPR are:
1. Generate a download file token.
2. Pass the token to the download endpoint in a query param and perform a standard http file dowload.

There are two ways to get the token. You can use the method using the ```fileTokens``` endpoint (Legacy) mentioned in the static docs, or use Swagger.
We will use Swagger.
> see [generateToken](#/2017/08/03/generateToken)

<pre><code class="javascript">
    /**
    * get the single use download token
    */
    function getFileToken(callback) {
        restClient.generateToken("DownloadFileTransferToken").then(() => {
            callback(null, token);
        }).catch(() => {
            callback(error);
        });
    },
</code></pre>

### Download the file
As mentioned previously, this is not an API that is part of the Swagger spec at the moment.
URL is created, and then a simple response stream piping to a file is used to save the file from that endpoint.

<pre><code class="javascript">
//restClient.js
    /**
    * use a simple file download pipe from response to save to downloads folder
    */
    function downloadFPR(token, callback) {                    
        const url = config.sscFprDownloadURL + "?mat=" + token + "&id=" + artifactId;
        try {
            let destFolder = __dirname + "/.." + config.downloadFolder;
            createDirectoryIfNoExist(destFolder);
            downloadFile(url, destFolder, filename, (err) => {
                if (err) {
                    callback(e);
                } else callback(null, destFolder + "/" + filename);
            });                        
        } catch (e) {
            callback(e);
        }

    }
</code></pre>