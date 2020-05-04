# ItemSynchronizer

This custom element synchronize items between two of your projects:

![screenshot](http://amend.cz/syncItems2.gif)

## Configuration

```json
{
    "serviceURL": "https://o4gdvtnha.execute-api.eu-central-1.amazonaws.com/default/syncFunction",
    "sourceProject": "1f33698f-a270-4b2d-90c5-9658a99c3140"
}
```

You need to link your synchronization service in the `serviceURL` parameter.

To set up `serviceURL` above, please follow [Working with sensitive data in custom elements](https://docs.kontent.ai/tutorials/develop-apps/integrate/working-with-sensitive-data-in-custom-elements).
In **Step 2: Configuring your Lambda function**, use the following keys and values in the **Environment variables** section:
  - ACCESS_CONTROL_ALLOW_ORIGIN:	<domain of this custom element>
  - HOST:	manage.kontent.ai
  - SOURCE_BEARER_TOKEN: <MAPI token from source project>
  - SOURCE_PROJECT: <source project ID>
  - TARGET_BEARER_TOKEN: <MAPI token from target project> 
  - TARGET_PROJECT:	<target project ID>
  
The code of such service:
A .Net Core lambda and php sample of such a requester:

```Javascript
const https = require("https");

/* ========Config Section======== */
const host = process.env.HOST;
const path = "/v2/projects/";
const sourceProject = process.env.SOURCE_PROJECT;
const targetProject = process.env.TARGET_PROJECT;
const accessControlAllowOriginValue = process.env.ACCESS_CONTROL_ALLOW_ORIGIN;

// Bearer token authentization
const sourceBearerToken = process.env.SOURCE_BEARER_TOKEN;
const targetBearerToken = process.env.TARGET_BEARER_TOKEN;
const sourceAuthorizationHeaderValue = `Bearer ${sourceBearerToken}`;
const targetAuthorizationHeaderValue = `Bearer ${targetBearerToken}`;

    
let endpoint = '';
let method = '';
let body_data = '';
let token = '';

const request = (queryStringParameters, headers, body) => {
    if (queryStringParameters) {
        switch (queryStringParameters.ep) {
            case "workflow":
                endpoint = path+targetProject+'/workflow';
                method = 'GET';
                body_data = '';
                token = targetAuthorizationHeaderValue;
                break;
            case "sourceitem":
                endpoint = path+sourceProject+'/items/'+queryStringParameters.id;
                method = 'GET';
                body_data = '';
                token = sourceAuthorizationHeaderValue;
                break;
            case "targetitem":
                endpoint = path+targetProject+'/items/codename/'+queryStringParameters.codename;
                method = 'GET';
                body_data = '';
                token = targetAuthorizationHeaderValue;
                break;
            case "sourcevariant":
                endpoint = path+sourceProject+'/items/'+queryStringParameters.id+'/variants/codename/'+queryStringParameters.culture;
                method = 'GET';
                body_data = '';
                token = sourceAuthorizationHeaderValue;
                break;
            case "targetvariant":
                endpoint = path+targetProject+'/items/codename/'+queryStringParameters.codename+'/variants/codename/'+queryStringParameters.culture;
                method = 'GET';
                body_data = '';
                token = targetAuthorizationHeaderValue;
                break;
            case "syncitemPOST":
                endpoint = path+targetProject+'/items';
                method = 'POST';
                body_data = body;
                token = targetAuthorizationHeaderValue;
                break;
            case "syncitemPUT":
                endpoint = path+targetProject+'/items/codename/'+queryStringParameters.codename;
                method = 'PUT';
                body_data = body;
                token = targetAuthorizationHeaderValue;
                break;
            case "syncvariant":
                endpoint = path+targetProject+'/items/codename/'+queryStringParameters.codename+'/variants/codename/'+queryStringParameters.culture;
                method = 'PUT';
                body_data = body;
                token = targetAuthorizationHeaderValue;
                break;
            case "movetoworkflow":
                endpoint = path+targetProject+'/items/codename/'+queryStringParameters.codename+'/variants/codename/'+queryStringParameters.culture+'/workflow/'+queryStringParameters.workflow;
                method = 'PUT';
                body_data = '';
                token = targetAuthorizationHeaderValue;
                break;
            case "publish":
                endpoint = path+targetProject+'/items/codename/'+queryStringParameters.codename+'/variants/codename/'+queryStringParameters.culture+'/publish';
                method = 'PUT';
                body_data = '';
                token = targetAuthorizationHeaderValue;
                break;
            case "sourcetypes":
                endpoint = path+sourceProject+'/types';
                method = 'GET';
                body_data = '';
                token = sourceAuthorizationHeaderValue;
                break;
            case "targettypes":
                endpoint = path+targetProject+'/types';
                method = 'GET';
                body_data = '';
                token = targetAuthorizationHeaderValue;
                break;
            case "newversion":
                endpoint = path+targetProject+'/items/codename/'+queryStringParameters.codename+'/variants/codename/'+queryStringParameters.culture+'/new-version';
                method = 'PUT';
                body_data = '';
                token = targetAuthorizationHeaderValue;
                break;
            case "sourcetaxonomies":
                endpoint = path+sourceProject+'/taxonomies';
                method = 'GET';
                body_data = '';
                token = sourceAuthorizationHeaderValue;
                break;
            case "targettaxonomies":
                endpoint = path+targetProject+'/taxonomies';
                method = 'GET';
                body_data = '';
                token = targetAuthorizationHeaderValue;
                break;
        }
    }
    
    headers["Host"] = host;
    headers["Authorization"] = token;
    headers["Accept"] = "application/json";
    headers["accept-encoding"] = "identity";
    headers["Content-Type"] = "application/json";
    
    
    const requestOptions = {
           host: host,
           path: endpoint,
           port: 443,
           method: method,
           headers: headers
        };
        
     
    return new Promise((resolve, reject) => {
        const req = https.request(requestOptions, response => {
            response.setEncoding('utf8');
            let data = '';
            response.on("data", chunk => {
                data += chunk;
            });
            response.on("end", () => {
                if (data) {
                    const dataObject = JSON.parse(data);
                    response.data = dataObject;
                }
                resolve(response);
            });
        });
        
        
        req.on("error", error => {
                reject(error);
            });
        if (body_data) {
            req.write(body_data);
        }
        req.end();
    }); 
     
};


exports.handler = (event, context, callback) => {
    
    if (event.headers.origin.toLowerCase() == accessControlAllowOriginValue.toLowerCase()) {
        const corsHeaders = {
            "Access-Control-Allow-Origin": accessControlAllowOriginValue
        };
    
        const repeatResponse = (response) => {
            let multiValueHeaders = {};
    
            for (const headerName in response.headers) {
                if (Array.isArray(response.headers[headerName])) {
                    multiValueHeaders[headerName] = response.headers[headerName];
                    delete response.headers[headerName];
                }
            }
    
            callback(null, {
                statusCode: response.statusCode,
                body: JSON.stringify(response.data),
                headers: {'Content-Type': 'application/json',...corsHeaders},
                multiValueHeaders: multiValueHeaders,
            });
        };
    
        const sendError = (error) => {
            callback(null, {
                statusCode: "400",
                body: JSON.stringify(error),
                headers: {'Content-Type': 'application/json',...corsHeaders},
            });
        };
    
       
        request(event.queryStringParameters, event.headers, event.body)
            .then((response) => {
                repeatResponse(response);
            })
            .catch(error => {
                sendError(error);
            });
        
    }
    else {
        callback(null, {
                statusCode: "403"
            });
    }
};
```

[![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/hzik/ItemSynchronizer/)



