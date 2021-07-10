---
slug: /blog/serverless-sinatra
title: "Serverless Sinatra"
description: Shows a way to run a Ruby Sinatra web services in AWS Lambda
quote: AWS Lambda allows you to host serverless functions written in almost any language. Let me show you how to use Ruby.
banner: serverless-ruby.png
date: 2017-06-10
---
At the moment the AWS Lambda service does not support ruby. It is however
possible to spawn another process from within one of the supported runtimes
e.g. nodejs. This post shows that it possible to run a rack based web
application in AWS Lambda.
<!-- More -->

# Show me the code!
The code that shows how to do this is at [serverless-ruby-sinatra-example](https://github.com/sheakelly/serverless-ruby-sinatra-example)

# How does it work?

<img src="serverless-ruby.png" class="center-block img-responsive padded" >

The project uses the [serverless framework](https://www.serverless.com) to
deploy a lambda function that responds to a AWS Api Gateway events. The lambda
function then spawns a process that execute some ruby code that invokes the
sinatra application via the rack webserver interface.

# Packaging the ruby code

AWS Lambda functions run within a container on an AWS EC2 instances. At the time
of writing this post the ruby runtime is unavailable. In order to make the ruby
runtime available I am using [travelling
ruby](http://phusion.github.io/traveling-ruby/) to package the ruby runtime and
all the gem dependencies as a zip file. These are included in the zip file the
serverless framework uploads to s3.

The code for packaging the ruby runtime and dependencies can be found in the
[Rakefile](https://github.com/sheakelly/serverless-ruby-sinatra-example/blob/master/Rakefile)

# shim.js

The shim.js script is a lambda function that handles API Gateways events by
spawning the rack_adapter.sh script and then passes the lambda `event` and `context`
objects to rack_adapter.rb as json text.

```sh
var proc = spawn('./ruby-bin/rack_adapter.sh', 
    [JSON.stringify(event, null, 2), JSON.stringify(context, null, 2)]
);
```

# rack_adapter.rb
The
[rack_adapter.rb](https://github.com/sheakelly/serverless-ruby-sinatra-example/blob/master/rack_adapter.rb)
simply converts the `event` and `context` structures into a rack environment
hash and then calls the sinatra application. The response is written to stdout.

# What are the pain points of this approach?

The main issues with this approach is that every time the lambda function is
invoked a new ruby vm is launched. This take about 700ms for this
really simple hello world style app. This is about 7 times slower then the
equivalent nodejs lambda.

Another limitation is that travelling ruby only supports up to ruby 2.2 and
there are limited pre built [native extension
gems](https://traveling-ruby.s3-us-west-2.amazonaws.com/list.html) available
that are compatible with Amazon linux. You could always build your application
and native dependencies on an Amazon Linux build agent to work around this.



