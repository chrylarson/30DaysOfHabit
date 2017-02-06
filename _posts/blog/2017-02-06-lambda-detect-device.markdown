---
layout: post
title:  "AWS Lambda Node.js Function to Detect Android or iOS device"
date:   2017-02-06 11:36:00 -0500
categories: blog tutorial
---
This is the fourth iteration of my AWS Lambda experiments:

[AWS Lambda function to return a tracking pixel]({{ site.baseurl }}{% post_url 2016-12-16-1x1_pixel_aws_lambda %})

[AWS Lambda Tracking Pixel with Elasticsearch]({{ site.baseurl }}{% post_url 2016-12-19-lambda-elasticsearch %})

[AWS Lambda, Kinesis Firehose and Elasticsearch]({{ site.baseurl }}{% post_url 2017-01-04-lambda-kinesis-firehose-elasticsearch %})

In this tutorial I demonstrate how to serve different HTML based on the device type. Detecting the device type allows me to serve iTunes links to iPhone and Google Play links to Android. In the previous posts I demonstrated how to set a cookie with a web token, log requests to Elasticsearch, and scale using Kinesis Firehose. 

Here’s the AWS Lambda function in node.js to read the User-Agent string and return different HTML for iPhone or Android devices. If the device cannot be determined I return HTML with links for each type of device.

### AWS Lambda Function to detect device type and return HTML
~~~~
'use strict';

exports.handler = function (event, context) {
    var userAgent = '';
    var htmlLink = '';
    var oss = 'unknown';

    if (event.hasOwnProperty('params') && event.params.hasOwnProperty('header') && event.params.header.hasOwnProperty('User-Agent')) {
        userAgent = event.params.header["User-Agent"];
    }

    if (userAgent.match("iPhone")) {        
        oss = "ios";    
    } else if (userAgent.match("Android")) {       
        oss = "android";
    }

    switch(oss) {
        case "ios":
            htmlLink = "<h1>Click the link below to subscribe to the Podcast</h1>"+ 
            "<a href='https://itunes.apple.com/us/podcast/internet-ballers-michael-pasha/id1108577427'>Subscribe on iTunes</a>"+ 
            ""+
            "";
            break;
        case "android":
            htmlLink = "<h1>Click the link below to subscribe to the Podcast</h1>"+ 
            "<a href='http://subscribeonandroid.com/www.internetballers.com/feed/podcast/'>Subscribe on Google Play</a>"+ 
            ""+
            "";
            break;
        default:
            htmlLink = "<h1>Click the link below to subscribe to the Podcast</h1>"+
            "<a href='https://itunes.apple.com/us/podcast/internet-ballers-michael-pasha/id1108577427'>Subscribe on iTunes</a>"+
            "<br >" +
            "<a href='http://subscribeonandroid.com/www.internetballers.com/feed/podcast/'>Subscribe on Google Play</a>"+
            "<br >" +
            "<a href='http://www.internetballers.com/feed/podcast/'>Subscribe to RSS</a>"+ 
            ""+
            "";
        }

    context.done(null, { "body": htmlLink });

};

~~~~

The setup is the same as it is in the previous tutorials.

I setup an AWS API Gateway in front of this Lambda function. To get the header data in the Lambda function I had to modify the *Integration Method*.

I setup a GET method for my API Resource. For the GET method I modified the *Integration Request* by setting the *Body Mapping Templates* to the third option, Never.

Then I added a Content-Type of application/json and for the template I selected Generate template: *Method request passthrough*. This creates a mapping that passes all the header data to the Lambda function.

### Integration Request, Body Mapping Templates, application/json, Method request passthrough mapping template
~~~~
##  See http://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html
##  This template will pass through all parameters including path, querystring, header, stage variables, and context through to the integration endpoint via the body/payload
#set($allParams = $input.params())
{
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
