---
layout: post
title:  "AWS Lambda Tracking Pixel with Elasticsearch"
date:   2016-12-19 10:43:00 -0500
categories: blog tutorial
---
Previously I wrote a small [AWS Lambda function to return a tracking pixel]({{ site.baseurl }}{% post_url 2016-12-16-1x1_pixel_aws_lambda %}). In this post I'm logging the request for the pixel to [Elasticsearch](https://www.elastic.co/). Elasticsearch is a search and analytics engine. Elasticsearch is paired with Kibana for creating graphs and visuals from the log data. 

I used [AWS Elasticsearch service](https://aws.amazon.com/elasticsearch-service/) to setup a t2.micro instance running Kibana and Elasticsearch. 

Once AWS finished launching the Elasticsearch instance I copied the Elasticsearch URL and pasted in my Lambda function below.

The Lambda function gathers the header data from the request along with the timestamp, stores it in Elasticsearch and returns a 1x1 pixel.

The data is in now searchable and viewable with Kibana.

The Lambda function is adapted from this [Sample Code from AWS](https://github.com/awslabs/amazon-kinesis-client-nodejs/tree/master/samples/click_stream_sample)

### Lambda Function to log event to Elasticsearch and return a 1x1 pixel
~~~~
'use strict';

var AWS = require('aws-sdk');
var path = require('path');

var esDomain = {
    region: 'us-east-1',
    endpoint: 'my-elasticsearch-url.us-east-1.es.amazonaws.com',
    index: 'events',
    doctype: 'event'
};
var endpoint = new AWS.Endpoint(esDomain.endpoint);
var creds = new AWS.EnvironmentCredentials('AWS');

var imageHex = "\x42\x4d\x3c\x00\x00\x00\x00\x00\x00\x00\x36\x00\x00\x00\x28\x00"+ 
"\x00\x00\x01\x00\x00\x00\x01\x00\x00\x00\x01\x00\x18\x00\x00\x00"+ 
"\x00\x00\x06\x00\x00\x00\x27\x00\x00\x00\x27\x00\x00\x00\x00\x00"+
"\x00\x00\x00\x00\x00\x00\xff\xff\xff\xff\x00\x00";

exports.handler = function(event, context) {
  console.log('Received event:', JSON.stringify(event, null, 2));
  var date = new Date();
  
  event.date = Date.now();
  var doc = JSON.stringify(event, null, 2);

    var req = new AWS.HttpRequest(endpoint);
    req.method = 'POST';
    req.path = path.join('/', esDomain.index, esDomain.doctype);
    req.region = esDomain.region;
    req.headers['presigned-expires'] = false;
    req.headers['Host'] = endpoint.host;
    req.body = doc;

    var signer = new AWS.Signers.V4(req , 'es');  // es: service code
    signer.addAuthorization(creds, new Date());

    var send = new AWS.NodeHttpClient();
    send.handleRequest(req, null, function(httpResp) {
        var respBody = '';
        httpResp.on('data', function (chunk) {
            respBody += chunk;
        });
        httpResp.on('end', function (chunk) {
            console.log('Response: ' + respBody);
            //return pixel
            context.succeed({ "body":imageHex });
        });
    }, function(err) {
        console.log('Error: ' + err);
        //return pixel
        context.fail({ "body":imageHex });
    });

};
~~~~

I setup an AWS API Gateway in front of this Lambda function. To get the header data in the Lambda function I had to modify the *Integration Method*.

I setup a GET method for my API Resource. For the GET method I modified the *Integration Request* by setting the *Body Mapping Templates* to the first option, When no template matches the request Content-Type header.

Then I added a Content-Type of application/json and for the template I selected Generate template: *Method request passthrough*. This creates a mapping the passes all the header data to the Lambda function.

### Integration Request, Body Mapping Templates, application/json, Method request passthrough mapping template
~~~~
##  See http://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html
##  This template will pass through all parameters including path, querystring, header, stage variables, and context through to the integration endpoint via the body/payload
#set($allParams = $input.params())
{
"body-json" : $input.json('$'),
"params" : {
#foreach($type in $allParams.keySet())
    #set($params = $allParams.get($type))
"$type" : {
    #foreach($paramName in $params.keySet())
    "$paramName" : "$util.escapeJavaScript($params.get($paramName))"
        #if($foreach.hasNext),#end
    #end
}
    #if($foreach.hasNext),#end
#end
},
"stage-variables" : {
#foreach($key in $stageVariables.keySet())
"$key" : "$util.escapeJavaScript($stageVariables.get($key))"
    #if($foreach.hasNext),#end
#end
},
"context" : {
    "account-id" : "$context.identity.accountId",
    "api-id" : "$context.apiId",
    "api-key" : "$context.identity.apiKey",
    "authorizer-principal-id" : "$context.authorizer.principalId",
    "caller" : "$context.identity.caller",
    "cognito-authentication-provider" : "$context.identity.cognitoAuthenticationProvider",
    "cognito-authentication-type" : "$context.identity.cognitoAuthenticationType",
    "cognito-identity-id" : "$context.identity.cognitoIdentityId",
    "cognito-identity-pool-id" : "$context.identity.cognitoIdentityPoolId",
    "http-method" : "$context.httpMethod",
    "stage" : "$context.stage",
    "source-ip" : "$context.identity.sourceIp",
    "user" : "$context.identity.user",
    "user-agent" : "$context.identity.userAgent",
    "user-arn" : "$context.identity.userArn",
    "request-id" : "$context.requestId",
    "resource-id" : "$context.resourceId",
    "resource-path" : "$context.resourcePath"
    }
}
~~~~
