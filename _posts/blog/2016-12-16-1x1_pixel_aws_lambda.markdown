---
layout: post
title:  "Serve a 1x1 Pixel using AWS API Gateway and Lambda"
date:   2016-12-16 09:01:00 -0500
categories: blog tutorial
---

Lambda Function to return a 1x1 Pixel. It took me a while to figure out how to return binary data from Lambda through the AWS API Gateway. Here's my Lambda function for returning a GIF image pixel.

### Lambda Function to return 1x1 pixel
~~~~
'use strict';
exports.handler = function(event, context) {

  var imageHex = "\x42\x4d\x3c\x00\x00\x00\x00\x00\x00\x00\x36\x00\x00\x00\x28\x00"+ 
  "\x00\x00\x01\x00\x00\x00\x01\x00\x00\x00\x01\x00\x18\x00\x00\x00"+ 
  "\x00\x00\x06\x00\x00\x00\x27\x00\x00\x00\x27\x00\x00\x00\x00\x00"+
  "\x00\x00\x00\x00\x00\x00\xff\xff\xff\xff\x00\x00"; 
  context.done(null, { "body":imageHex });

};
~~~~

Then in the *Method Response* for the *API Gateway* I added Content-Type to the *Response Header*.

Then in the *Integration Response* I map Content-Type to the constant string, 'image/gif' (single quotes).
In the *Integration Response* -> *Body Mapping Templates* I added a template for image/gif.

### image/gif Body Mapping Template
~~~~
#set($inputRoot = $input.path('$'))
$inputRoot.body
~~~~

Now I can call the pixel from my webapges.

~~~~
<img src="https://api-gateway-url/stage/pixel.gif" />
~~~~