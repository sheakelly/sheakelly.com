---
slug: /blog/debugging-lambda-limits
title: "Debugging lambda limits"
description: How to debug the memory, file descriptor and tmp storage limits
quote: AWS Lambda has a number of limits we need to keep an eye on. This post shows how you can debug some of the limits.
banner: debugging.png
date: 2019-05-02
---

# Lambda limits

There a number of limits imposed by AWS when using the AWS Lambda service.
They are documented at https://docs.aws.amazon.com/lambda/latest/dg/limits.html

On a recent project that involved generating and manipulating pdf files I ran into
some of these limits. This is how I debugged and corrected the errors.

## File descriptors

When trying to convert thousands of files to pdfs we started to see errors like:

```
Error: spawnSync /bin/sh EMFILE
    at _errnoException (util.js:1022:11)
    at spawnSync (child_process.js:579:20)
    at execSync (child_process.js:635:13)
    at Object.module.exports.convertFileToPDF (/opt/nodejs/node_modules/@shelf/aws-lambda-libreoffice/src/index.js:26:16)

Error: getaddrinfo EMFILE ssm.ap-southeast-2.amazonaws.com:443
    at Object._errnoException (util.js:1022:11)
    at errnoException (dns.js:55:15)
    at GetAddrInfoReqWrap.onlookup [as oncomplete] (dns.js:92:26)
```

After googling a bit it turns out that these errors occur when the system runs out of available file descriptors.
File descriptors on Amazon Linux include open files and open sockets. This mean that we were either
not closing files or leaving sockets open. To understand what was happening I needed to find out
what file descriptors existed for the node process that runs the lambda function.

To see the current file descriptor you can run

```ts
import {exec} from 'child_process';
import {promisify} from 'util';
import process from 'process';

const execP = promisify(exec);

const {stdout} = await execP(`ls -l /proc/${process.pid}/fd`);

console.log(stdout);
```

A sample of the ouput looks like this:

```
lrwx------ 1 sbx_user1061 485 64 May  1 05:27 0 -> /dev/null
l-wx------ 1 sbx_user1061 485 64 May  1 05:27 1 -> pipe:[107780]
l-wx------ 1 sbx_user1061 485 64 May  1 05:27 10 -> /opt/amazon/asc/worker/sb_log/sb10.log
lrwx------ 1 sbx_user1061 485 64 May  1 05:27 11 -> socket:[106947]
lrwx------ 1 sbx_user1061 485 64 May  1 05:27 12 -> socket:[107845]
lrwx------ 1 sbx_user1061 485 64 May  1 05:28 13 -> socket:[170987]
lrwx------ 1 sbx_user1061 485 64 May  1 05:27 15 -> socket:[107777]
lrwx------ 1 sbx_user1061 485 64 May  1 05:28 16 -> socket:[170989]
lrwx------ 1 sbx_user1061 485 64 May  1 05:27 18 -> socket:[107779]
lrwx------ 1 sbx_user1061 485 64 May  1 05:28 19 -> socket:[170991]
...
```

In our case it turned out that we had a number of sockets opened and after every invocation the number of opened sockets increased. We had a leak!
Turns out we were creating a new db connection pool
on every lambda invocation. As lambda will reuse the underlying container we quickly hit the 1024 file descriptor limit

## Memory

To see the amount of memory used by your lambda function simply look at the `Max Memory Used` value in the lambda REPORT

```
REPORT RequestId: cdc0715c-147b-5e5f-8aa0-241f214ce905	Duration: 756.36 ms	Billed Duration: 800 ms 	Memory Size: 3008 MB	Max Memory Used: 698 MB
```

## /tmp storage

When working with the lambda we are given 512Mb of space under /tmp to write files to.
As lambda will re-use a function when it can we need to make sure we clean up /tmp ourselves.
To detect if you are correctly cleaning up /tmp you can simply run the `du` command to see
if the space used is increasing after every lambda invocation and is leaking storage.

```ts
import {exec} from 'child_process';
import {promisify} from 'util';

const execP = promisify(exec);

const {stdout} = await execP(`du -sh /tmp`);
console.log(stdout);
```

The output looks like this:

```
24K	/tmp
```
