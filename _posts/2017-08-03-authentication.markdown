---
title:  "Authentication"
date:   2017-08-03 11:03:15 -0700
---
## Authenticating with SSC
The Swagger spec enforces security schemas for any API call.
The call order needed to successfully authenticate with SSC is as follows:

1. Call to obtain a unified login token using basic http auth.
2. Make all subsequent calls to SSC passing that token.


The ```initialize()``` function in the restClient.js takes care of that.
> If you are not familar with the async library please read [this](https://caolan.github.io/async/docs.html#waterfall){:target="_blank"}. The waterfall function basically runs a set of async functions sequentially. The ```callback``` method tells the next function to start.

### Getting the token
```api``` is the reference to the Swagger-generated object.
We call the ```createAuthToken``` passing required parameters. In the Promise resolving callback (```then```), we take the token from the resolve and pass on to the next waterfall handler. 
> The first parameter in the callback is reserved for error objects.

<pre><code class="javascript">
//restClient.js
...
    async.waterfall([
        //get the token from SSC using basic http auth
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
        ...
</code></pre>

### Using auth to test API
Now that we have the token, we will use it to call an API function that SSC has for the purpose of testing connectivity and correctness.
A ```"list-features"``` resource action is done on the feature-controller. The results just checks for an error and passes the token along.

<pre><code class="javascript">
//restClient.js
...
    },
    function heartbeat(token, callback) {
        //We have token, so try it out.
        //SSC has a "ping" endpoint called features that is very quick (no db or processing)
        //getClientAuthTokenObj - see top of snippet, returns an object that instructs Swagger to create a token header
        //That is, it will end up sending "Authorization FortifyToken [The token from previous function]
        api["feature-controller"].listFeature({}, getClientAuthTokenObj(token)).then((features) => {
            callback(null, token);
        }).catch((error) => {
            callback(error);
        });
    }
        ...
</code></pre>


### Waterfall done
Once the waterfall of async functions is done, we reach the onDoneHandler. All this method does is resolve the Promise that the ```initialize()``` method returns. That way, anyone who calls it can use ```then()``` and ```catch()```.

<pre><code class="javascript">
        ], function (err, token) {
            //This is the global waterfall handler called after all successful functions executed or first error
            if (err) {
                reject(err);
            } else {
                that.token = token; //save token
                resolve("success");
            }
        });

    }).catch((err) => {
        reject(err);
    });

})
</code></pre>
