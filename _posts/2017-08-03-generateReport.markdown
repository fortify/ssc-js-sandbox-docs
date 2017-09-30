---
title:  "Report: Generate, Track & Download"
date:   2017-08-03 11:01:12 -0700
type: "spec"
command: "npm run generateReport"
commandSub: "sscAPIBase, username, password, downloadFolder"
---
## Overview
This doc will show how to do the following:
1. Schedule a report for processing
2. Track the report until processing is complete
3. Download the report.


## Schedule a report for processing
Generating a report consists of two steps in this tutorial. 
1. Find the report definition id based on name.
2. Generate a DISA STIG report.

> The code can be generalized even further to support other report types, but for the sake simplicity we will pass some parameters specifically to "DISA STIG". For example, ```"type": "ISSUE"```.

### Getting the report definition Object
In the following code, we use the "q" parameter supported by many API calls in SSC. This parameter allows narrowing down of results based on values inside properties of the report definition. Notice the ```q: "name:" + stigDefName```. This will eventually get translated to 
http://[ssc]/api/v1/reportDefinitions?q=name:"DISA STIG"

<pre><code class="javascript">
//restClient.js
    /**
    * Query the server for the definition id based on the name of the report definition uploaded to SSC (the default is usually "DISA STIG")
    */
    function getDefinition(callback) {
        restClient.api["report-definition-controller"].listReportDefinition({ q: "name:" + stigDefName },
            getClientAuthTokenObj(restClient.token)).then((result) => {
                if (result.obj.data && Array.isArray(result.obj.data) && result.obj.data.length === 1) {
                    callback(null, result.obj.data[0]);
                } else {
                    callback(new Error("Could not find " + stigDefName + " report definition on the server, look in the report definition in the admin section"));
                }
            }).catch((err) => {
                callback(err);
            });
    },
</code></pre>

### Report definition
The report definition object is a generic description of what parameters *this specific report* accepts.
For example, DISA STIG accepts parameters of type SINGLE_SELECT_DEFAULT with optional values such as "ExternalListGUID" that we can pass in either the DISA STIG 4.3 or DISA STIG 4.2 or other options.

<pre><code class="javascript">
    ....
    "name": "DISA STIG",
    "guid": "DISA_STIG",
    "id": 13,
    "parameters": [
        {
            "id": 40,
            "name": "Options",
            "type": "SINGLE_SELECT_DEFAULT",
            "description": "DISA STIG Options",
            "identifier": "ExternalListGUID",
            "reportDefinitionId": 13,
            "paramOrder": 0,
            "reportParameterOptions": [
                {
                    "id": 5,
                    "displayValue": "DISA STIG 4.3",
                    "reportValue": "A0B313F0-29BD-430B-9E34-6D10F1178506",
                    "defaultValue": true,
                    "description": "DISA STIG 4.3",
                    "order": 0
                },
                {
                    "id": 6,
                            "displayValue": "DISA STIG 4.2",
        ...
</code></pre>                            

#### Creating the basic request data
First step is to populate some basic parameters
<pre><code class="javascript">
//restClient.js
    const requestData = {
        "name": name, "note": notes, "type": "ISSUE", "reportDefinitionId": definition.id, "format": format,
        "project": { "id": projectId, "version": { "id": 3 } }, "inputReportParameters": []
    };
</code></pre>

#### Filling in parameter values
Iterate the parameters in the definition and map each one to a param to be sent to the server with value selected.

> Other report definitions may have other parameter types.

<pre><code class="javascript">
//restClient.js
     //go through report definition parameters and fill in some data
    requestData.inputReportParameters = definition.parameters.map((paramDefinition) => {
        const param = {
            name: paramDefinition.name,
            identifier: paramDefinition.identifier,
            type: paramDefinition.type
        };
        switch (paramDefinition.type) {
            case "SINGLE_SELECT_DEFAULT":
                //find the report param option that has order 0 (most recent)
                param.paramValue = paramDefinition.reportParameterOptions.find(option => option.order === 0).reportValue;
                break;
            case "SINGLE_PROJECT":
                param.paramValue = versionId
                break;
            case "BOOLEAN":
                 //send "false" for all values for the sake of example and speed of processing.
                //this could be turned into another switch case based on parameter identifier
                //such as SecurityIssueDetails or IncludeSectionDescriptionOfKeyTerminology
                param.paramValue = false;
                break;
        }
        return param;
    });
</code></pre>

#### Create report on server (Schedule for processing)
<pre><code class="javascript">
//restClient.js
    restClient.api["saved-report-controller"].createSavedReport({
        resource: requestData
    }, getClientAuthTokenObj(restClient.token)).then((resp) => {
        callback(null, resp.obj.data);
    }).catch((error) => {
        callback(error);
    });;
</code></pre>

## Track report status
The following example shows how to track the status of processing of this report on the backend.
Report generation is an asynchronous process and therefore requires tracking.

<pre><code class="javascript">
//generateReport_spec.js
   /**
   * Poll the report created in previous test for processing status
   */
  it('poll for status ', function (done) {
    if (!savedReportEntity) {
      return done(new Error("previous report generation failed"));
    }
    /*
    * callback for setTimeout. Will use restClient to get the report status and then, based on returned status, either set another timer or stop execution.
    */
    function checkStatus() {
      const timeoutSec = 2;
      restClient.getSavedReportEntity(savedReportEntity.id).then((savedReport) => {
        savedReportEntity = savedReport; //update for next test
        switch (savedReport.status) {
          case "PROCESSING":
            console.log(`${savedReport.name} id(${savedReport.id}) still processing! will check again in ${timeoutSec} seconds`);
            setTimeout(checkStatus, timeoutSec*1000);
            break;
          case "ERROR_PROCESSING":
            let msg = `${savedReport.name} id(${savedReport.id}) failed!`;
            console.log(chalk.red(msg));
            done(new Error(msg));
            break;
          case "SCHED_PROCESSING":
            console.log(`${savedReport.name} id(${savedReport.id}) still scheduled! will check again in ${timeoutSec} seconds`);
            setTimeout(checkStatus, timeoutSec*1000);
            break;
          case "PROCESS_COMPLETE":
            console.log(chalk.green(`${savedReport.name} id(${savedReport.id}) completed successfully`));
            done();
            break;
        }
      }).catch((err) => {
        let msg = `${savedReport.name} id(${savedReport.id}) failed! ` + err.message;
        console.log(chalk.red(msg));
        done(err);
      });
    }
    checkStatus();
  });
</code></pre>

## Download report
Downloading the report is a simple piping of the http response stream to a file.
Similar to the upload FPR calls, the download API URL is *not part of* the REST API Swagger code generation.

The async.waterfall sequence starts with generating a one-time file download token.
<pre><code class="javascript">
//restClient.js
...
    downloadReport(reportId, filename) {
        const restClient = this;
        return new Promise((resolve, reject) => {
            async.waterfall([
                /**
                * get the single upload token
                */
                function getFileToken(callback) {
                    restClient.generateToken("ReportFileTransferToken").then((token) => {
                        callback(null, token);
                    }).catch((error) => {
                        callback(error);
                    });
                },
</code></pre>

The second part of the sequence is a call to download the file. As mentioned previously, the file download in SSC is not part of the Swagger code generation.
URL is created, and then a simple response stream piping to a file is used to save the file from that endpoint.
<pre><code class="javascript">
//restClient.js
    /**
     * use a simple file download pipe from response to save to downloads folder
    */
    function downloadReport(token, callback) {
        //download the file - for more info on this in SSC navigate to [SSC URL]/<app context>/html/docs/docs.html#/fileupdownload
        const url = config.sscReportDownloadURL + "?mat=" + token + "&id=" + reportId;
        try {
            let destFolder = __dirname + "/.." + config.downloadFolder;
            downloadFile(url, destFolder, filename,  (err) => {
                if (err) {
                    callback(e);
                } else callback(null, destFolder+ "/" + filename);
            });
        } catch (e) {
            callback(e);
        }

    }
    ], function onDoneDownload(err, dest) {
</code></pre>                
