---
layout: post
title:  "SSL with node.js, express and socket.io"
date:   2013-02-18 12:51:00 -0500
categories: blog tutorial
permalink: /blog/ssl-nodejs-express-and-socketio/
---
Using HTTPS with express is almost as simple as replacing `require('http')` with `require('https')`. For my application I am using a self-signed certificate that I created after following the directions on [Nate Good's blog](http://blog.nategood.com/client-side-certificate-authentication-in-ngi). I placed the certs in a folder called cert.

I initialized the variables in preparation for two servers following [this Stackoverflow question](http://stackoverflow.com/questions/7450940/automatic-https-connection-redirect-with-node-js-express). One server for https listening on 443 and another for http listening on port 80. Then I wrote the http server to forward all requests to the https server. So that node doesn't run as root I edited the iptables to map port 80 to 8081 and port 443 to 8080.

~~~~
var app = express()
  , httpapp = express()
  , fs = require('fs')  
  , options = {
    key: fs.readFileSync('./cert/client.key'),
    cert: fs.readFileSync('./cert/client.crt'),
    requestCert: true
}
  , http = require('http').createServer(httpapp)
  , server = require('https').createServer(options, app)  
  , io = require('socket.io').listen(server);
~~~~

The following code forwards all requests for HTTP to HTTPS.

~~~~
httpapp.get('*',function(req,res){  
    res.redirect('https://127.0.0.1:8080'+req.url)
})

server.listen(8080);
http.listen(8081);
~~~~

Then on the client side I had to setup socket.io to use the secured port as well.

~~~~
var socket = io.connect('https://127.0.0.1:8080', {secure: true});
~~~~

The process was first to obtain or create an SSL Certificate. Then create an express server that requires 'HTTPS'. Finally, setup the client JavaScript to reply on the HTTPS secure port.

EDIT 7/24/13: Also check out the [documentation for HTTPS](http://nodejs.org/api/https.html).