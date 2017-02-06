---
layout: post
title:  "AWS Lambda, Kinesis Firehose and Elasticsearch"
date:   2017-01-04 10:43:00 -0500
categories: blog tutorial
---
This is the third iteration of my tracking pixel experiment:

[AWS Lambda function to return a tracking pixel]({{ site.baseurl }}{% post_url 2016-12-16-1x1_pixel_aws_lambda %})

[AWS Lambda Tracking Pixel with Elasticsearch]({{ site.baseurl }}{% post_url 2016-12-19-lambda-elasticsearch %})

The Lambda to Elasticsearch function works well, but what happens when there is a traffic spike?

In this tutorial I add [Kinesis Firehose](https://aws.amazon.com/kinesis/firehose/) so my app scales quickly. The incoming requests are written to Kinesis Firehose and later sent to Elasticsearch and also backed up in S3.

First, I used [AWS Elasticsearch service](https://aws.amazon.com/elasticsearch-service/) to setup a t2.micro instance running Kibana and Elasticsearch.

Second, I setup [Kinesis Firehose](https://aws.amazon.com/kinesis/firehose/) to publish to S3 and the Elasticsearch cluster I just created.

Third, I setup API Gateway as I did in the previous blog posts.

Finally, I updated the lambda function to publish to Kinesis Firehose instead of Elasticsearch.

### Lambda Function to log events to Kinesis Firehose
~~~~
'use strict';

var AWS = require('aws-sdk');
var setRegion = process.env['us-east-1'];
AWS.config.update({ region: 'us-east-1' });
var path = require('path');

var fh = new AWS.Firehose({
		apiVersion : '2015-08-04',
		region : setRegion
	    });

var imageHex = "\x42\x4d\x3c\x00\x00\x00\x00\x00\x00\x00\x36\x00\x00\x00\x28\x00" +
    "\x00\x00\x01\x00\x00\x00\x01\x00\x00\x00\x01\x00\x18\x00\x00\x00" +
    "\x00\x00\x06\x00\x00\x00\x27\x00\x00\x00\x27\x00\x00\x00\x00\x00" +
    "\x00\x00\x00\x00\x00\x00\xff\xff\xff\xff\x00\x00";
var es_data = {};

exports.handler = function (event, context) {
    // Add date for Elasticsearch index
    var d = new Date();
    es_data.date = d.toISOString();
    es_data.context = event.context;
    es_data.params = event.params;

    var jsonDoc = JSON.stringify(es_data);

    fh.putRecord(
        {
            DeliveryStreamName: 'kinesis-firehose-stream-name',
            Record: { Data: jsonDoc }
        },
        function (err, data) {
            if (err) {
                console.log(err);            // an error occurred
            } else {
                console.log(data);           // successful response
            }
            context.done(err, { "body": imageHex });
        });

};
~~~~
