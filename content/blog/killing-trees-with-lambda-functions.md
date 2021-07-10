---
slug: /blog/killing-trees-with-lambda-functions
title: "Killing Trees with Lambda Functions"
description: How we built a serverless solution on AWS to significantly reduce the time it takes to print client progress notes
quote: Here are some key design decisions we made and patterns we used to build a serverless solution on AWS. It is now easy to print large quantities of client progress notes.
banner: paper-piles.jpg
date: 2019-07-10
---

# The business problem

My organisation provides various services to thousands of clients across Australia. This includes children in foster care,
people with disabilities and the elderly. As part of providing support services our care workers must records progress
note for all interactions with our clients. The progress notes are captured in one of our core systems and reside in a SQL Server database. Unfortunately
access to this system is difficult especially from remote locations. As a result rather than entering text records most of the progress notes are stored
as other file type e.g. Word documents, images, pdfs and Outlook.

At various times our support workers need to provide the contents of the progress notes to various exteral entities. This
includes government departments, other origanistations taking over care or to the client themselves when they transition
out of our care. The result is often __weeks__ of work manually downloading and printing each individual
documents then collating them into what can be 1000's of printed and bounded pages. This sort of effort prevents the
support workers from doing the high value client work they enjoy. They get bogged down manually printing documents.

After getting a reasonably understanding of the buiness problem we quickly realised we were basically being asked
to build a tool to convert the 1000's of progress notes stored as text and their associated attachments into a large
pdf file that could be easily sent to the printer. This prompted a few key questions: 

1. What software packages can we used to generate pdf files?
2. What are the file types we need to support?
3. What about nested file types e.g. zip files and email files with attachments?
4. If we generate a large pdf can we deliver that to the third party rather than wasting all this paper?
5. How will we access the existing systems SQL server database?

# Can we really use a serverless approach for this?

During our initial discussions about the technical solution a few solution alternatives came to mind. 
We set out brainstorming what the key technical risks were for each solution what technical spikes
we needed to do to reduce the technical risk of each solution.

## Team skills and background

Some notes about the team:
- All senior developers
- All previously build at least one application that involved using AWS services including AWS Lambda
- Our organisation has been building applications on AWS for the last 2 years.

Given our previous experience and our positive past experiences with AWS Lambda we had to find a reason 
to NOT use a serverless approach.

## Alternatives

### Container based
The container based approach seemed like a good option. We were unsure how long it would take
to convert the various file types to pdf files. We were also unsure if we would be bound to 
the performance of the CPU. 

The container approach would give us maximum flexibility with regards to how we generate pdf files
allowing to use any software package and provide a longer amount of time to run a conversion job

### AWS Lambda 
Using AWS Lambda mean't we would need to deal with the limitations of lambda. The key concerns were:
- Maximum excution time of 15 minutes
- Can we run custom linux binaries in the lambda execution environment
- Would the conversion tools need more memory that we can allocate to a lambda function

### Hybrid 
This approach would involve using AWS lambda to host a REST Api to be used by the single page application
and using either ECS, EKS or the newer Fargate service to produce the pdf files. This felt like a
good options but also the more complex because we would have to either stand up a docker cluster (not something for the faint hearted)
or learn how to use AWS Fargate.

## The application architecture
![architecture](print-all-aws-arch.png)

## How we chose serverless
We ran an experiment (aka Spike) to see if we
could convert some attachment files to pdf files using the libreoffice binaries in AWS Lambda. This turned out 
to be easier than we thought and has been solved by this awesome project https://vladholubiev.com/serverless-libreoffice/.

We were able to build a simple lambda function that would:
1. Download a file from s3
2. Convert it to a pdf using libreoffice
3. Put the pdf back in s3
We could then test it out on the various known file types.

It turn out the conversion process for a single file was usually less that 1 second. 
The ease with which we were able to get this working gave us enough confidence to start with
the AWS Lambda approach. I think also in the back of our minds we knew we could fall back on
one the hybrid container based alternatives if we needed to.

We also tested out [pdftk](https://www.pdflabs.com/tools/pdftk-the-pdf-toolkit/) to ensure that we could take
a number of individual pdf files and merge them into a one pdf.

## Some thoughts on serverless
Why does serverless matter? Well it allows us to build projects using infrastructure pieces that others have built
without the overhead of building and managing the infrastructure ourselves. Serverless like micro services, agile and AI has 
become a marketing buzzword. There are conflicting
definitions and these terms are adopted by product vendors to push their products. It is important to
consider what serverless is. There are a number of factors that should be considered when we judge the
serverlessness of a product or service and each product or service will sit somewhere on the scale of 
serverful to serverless.

### Pay per use
We only pay for what we use. When using AWS Lambda you pay for how long your lambda function executes. It is
a serverless compute service. SQS is another example. You pay for the quantity of messages passing through the 
queuing infrastructure. We don't pay for under utilised servers sitting idle. Most software
as a service products (SaaS) fall down here where you need to provision a new tenancy at a monthly rate even if
it is unused.

### Auto scaling
As traffic or usage of the service increases or decreases it should automatically adjust itself to handle the new
traffic pattern. The key is that the decision to scale is not made by our developers or operations staff it
just happens based on the rules of the specialist team managing the service for us.

### Minimal infrastructure to manage
No servers to manage. Someone else does it who specialises in running racks of servers or container infrastructure. No data centre
refreshes and no need for seperate a operation team. The development team (that should of course include some operations skills)
monitor and support the services they own and maintain. They treat what they build as a product that will evolve over time and not as
a one off project to be handed over to someone else to look after.

## Lambda function design
The are a number of limits that we need to be aware of when using the AWS Lambda services. They are documented at
https://docs.aws.amazon.com/lambda/latest/dg/limits.html. The 

After spiking out and prooving that the we could produce and combine pdf documents within the memory limits our next
concern was function timeout limit. Would fifteen minutes be long enough for longest job to run. Maybe. We decided
to design our way out of this limitation. Given lambda is best suited to lots of short lived functions running concurrently
we split the processing of a job into a number of individual functions:

### Functions:
#### app function
This function is responsible for accepting http events from API Gateway and forms the REST api for application.
A user can find a client by their unqiue identifier and then request a job to be created for that client. A 
`POST /api/client/:clientId` will validate the request, verify the user has access to be allowed to see the clients
details and asynchronously invoke the `jobItemCreator` function.

#### jobItemCreator function
Given a client number this function queries the SQL Server database for all the progress notes and thier attachments and then produces
a list of `JobItem`s. These are then stored in dynamodb and placed on an sqs queue for processing by the `jobItemProcessor`
function.

#### jobItemProcessor function
This is the probably the most complex function in the system. It receives `JobItem` messages from a queue then
depending if the item is a progress note or attachment grabs the appropriate text or binary data from SQL Server,
stores it in `/tmp` directory then either converts it to pdf using `libreoffice` or
in the case of an outlook `.msg` file converts it to an `.eml` file extracts the email attachments and
recursively processes the results by adding new job items onto the queue. After all this it updates the `JobItem` 
in dynamodb to indicate the result and updates the overall status for the `Job`.

When all the `JobItem`s are processed we end up with thousands of pdf files sitting in s3 ready to be bundled.

#### jobBundler function
The `jobBundler` is hooked up to the event stream of DynamoDB and looks for events with a job status of
`ReadyToBundle`. It then queries dyanmodb for all the job items and groups them in quarters of the year. e.g. 2018-Q1,
2018-Q2, 2018-Q3 etc. As we limited to storing 512MB in `/tmp` we also split the quarterly file in up to 200MB chunks as needed.
This gives us enough space to download the files from s3 to `/tmp` and enough space to produce the output of bundling.
`Bundle` records are then stored in dynamodb and placed on another SQS queue to be processed by the `bundleProcessor`.

#### bundleProcessor function
the `bundleProcessor` consumes bundles messages off a SQS queue. The bundle data structure list all the pdf files that make up
a bundle. It then builds a command and executes `pdftk` to combine many pdf files into a single bundled pdf. 
The bundle is uploaded to s3 and the job status is updated to `Complete` when all bundles are processed.

### Concurrency limitation
The `jobItemProcessor` is the function that executes most often. Each job consists of hundreds or
thousands of job items that can be processed concurrently. The main limitation on concurrency is the number
of database connections to our SQL Server database. Each lambda function when is starts create a new connection
and keeps it open for the for lifetime of the function exection context i.e. container. As we are using the node js runtime we chose the 
[knex](https://knexjs.org/) library and set its connection pool to have `{min: 1, max: 1}` connections.
We then set `reservedConcurrency: 5` for the lambda to ensure only 5 connections are created at any time. If we
need to scale up we could increase this value.

We are also limited to the overall account level concurrency per region for a lambda in 1000. We could request
an increase if we needed it.

### AWS Step Functions - No Thanks!
One AWS service we considered in our intial design was AWS Step Fuctions. This service allows you to orchestrate
seperate lambda functions into workflows. This seemed like it could fit given we were essential modelling a
scatter and gather processing pattern whereby we split a job up into a number of descrete job items and
combine the results at the end. 

After looking at the documentation we quickly decided that rather than encode the logic of the system into
a json document we would see if see could get by without it. Also mean't that we did not need to learn
yet another service and dsl style language. It also bought back bad memories of Microsoft Biztalk and Windows
Worflow Foundation. For our team it seemed simpler to just build and orchestration logic in typescript.
That way we can use the same set of testing tools we would use for any typescript module.

## Language choices
### Typescript
We chose to use typescript to write the lambda functions. We had used it before and were not put off by the
additional syntax required to define types and type annotations. In a previous life most of us have been Java or C# developers. 

Typescript allowed us to define a few data types to work with across the application. Here is what a `Job`
looks like:

```js
interface Job {
  id: JobId
  status: JobStatus
  clientId: clientId
  ...
}
```
We also have `JobItem`, `Bundle` and some other supporting types. This gives us compile time checking so as we altered
the structures over time the compiler tells us were the errors are. There is something nice about having 
a compiler tell you were there are problems before you deploy and run the code.

### elm
For the frontend we went head first into elm. The team already had one member with elm experience and the rest had
a solid understanding of [The elm architecture](https://guide.elm-lang.org/architecture/) from using redux and react.
There is definately a learning curve picking up a new language that is also dsl for single page applications. Elm
is a pure functional language. It has a haskell style syntax which can be a shock for those use to C-style languages.
That said our developlers picked it up quickly.

Our application consists of 3 pages. A jobs page, results pages and page to show any processing errors that have
occurred. elm has built in support for client side navigation. It does however get complicated when you need
to manage multiple pages and manage their state. This is an aspect of the elm architecture that can 
introduce a bit of boilerplate code for each new page added. We opted to have a single `Model` type
to hold all the data for the application. This unforntunatley makes it less obvious which view/page uses what
parts of the model. We may want to consider refactoring this to be more in line with Richard Feldmans's 
[Realworld Example App](https://github.com/rtfeldman/elm-spa-example). With a single `Model` you avoid the boilerplate
but the tradeoff is that you are not making [impossible states impossible](https://www.youtube.com/watch?v=IcgmSRJHu_8).
The elm community and its thought leader [Evan Czaplicki](https://twitter.com/czaplic)
recommend starting with what is simple and works and letting the problem dictate the design rather than setting
up a large framework upfront and making big upfront mistakes. This is a refreshing approach!


![Main Screen](./MainScreen.png)
*The main screen of the elm application whilst a job is processing*
</br>
</br>
</br>
![Results Screen](./ResultsScreen.png)
*The results of processing. Some big pdf files here!*

## Functional Programming
Our overall solution generally uses a functional style. The front-end is elm which forces the point. The lambda functions
are built with typescript with the `strict: true` option enabled. This give us some level of type safety and will
not allow us to unintentionally use `any` types. We also have made use of some functional programming libraries and
techniques. We are using some functions from [ramda](https://ramdajs.com/) in particular the [curry](https://ramdajs.com/docs/#curry)
to allow us to use partial application where appropriate. We also use higher order functions as a way of isolating
a function from having to know too much about other effectful parts of the system. It gives us a way to control the 
returned value from in a unit test by simply passing in a different function implementation of the higher order function. 

This is an example of a curried expressjs handler function whose first argument is a function that allows us to get a Job for an
id. This express handler does not care how or when it comes from. This is an alternative to jest module mocking.

```js
export const getJobResults = curry(
  async (
    getJobById: GetJobByIdFn,
    req: RequestWithUser,
    res: Response
  ): Promise<any> => {
  ...
  const job = getJobById(request.params.jobId)
  ...
```

## Serverless Framework
The [serverless framework](https://serverless.com/) makes it easier to deploy lambda based solution to
AWS. It has a bunch of plugins and has some great features like [variables](https://serverless.com/framework/docs/providers/aws/guide/variables/)
that make it more useful than using plain cloud formation templates. The aws provider essentially generates
a cloudformation template that you can add your own resources into e.g. a dynamo table. Plugins can
be used to augment the cloudformation template too. There are alternative like [SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
that we could use. But we know the serverless framework quite well. We even use it deploy stacks with no lambda
functions! Its command line interface is fairly easy to use and it fits well into our node js and javascript tool chain.

## Everything is a cloudformation stack
A pattern that we have experimented with recently is to try and make all deployed resources part of a cloudformation
stack. Obviously this is easier for the resource types that AWS supports natively. We try to create a custom resource type
for the deployed resources that either do not have an AWS provided resource type (come on AWS why does cloud formation lag behind
service releases) or for resources that are not deployed to AWS e.g. SaaS products.

Why do this? In sort it allows us to clean up old unused resourses. We use a feature branch based git workflow. 
We branch off master make our changes and deploy the new version into our test AWS account. 
The beauty of the cloud is we can spin up as many copies of an application as we like.
We then run automated and manual testing in this environment and merge to master to release. So now that
we are finished our new feature and done with our feature branch [batman](https://github.com/nib-health-funds/cfn-batman), well a fork of
batman comes along and deletes the stacks of our branched code. With everything as a cloudformation resource
we can be confident that we are not leaving test versions of things hanging around. As most SaaS products
provide apis creating a custom resources it relatively easy.

## Lerna monorepo
[Lerna](https://lerna.js.org/) is a tool that allows us to have multiple npm modules within a single git repository.
Before using lerna we would create a project group in gitlab with many repositories within it. This lead to few
problems solved by lerna:
- Duplication of project configuration files e.g. typescript and jest config
- Merge requests that span multple repositories requiring awkward release processes
- Additional `yarn link` steps in development workflows

In general I find it a controversial tool as it goes against the node.js culture of lots of small independent single purpose
modules. Lerna still allows you to have lots of small modules just in the one repo. The way we use
it for building projects is to have fewer modules and break them up as it makes sense. This is what our
repository stucture looks like:
```
print-all/ (project root)
|--packages/
|  |--api/
|  |--client/
|  |--cloudfront/
|  |--e2e-tests/
|  |--libreoffice-layer/
|  |--msgcovert-layer/
|  |--pdtk-layer/
|  |--tsconfig.js
|--jest.config.test
|--lerna.json
|--tslint.json
|--package.json

```
Some folders and files have been excluded but the general pattern is that each package has its own
`package.json` and is deployed independently and there is some config shared across them. As we
are using elm for our client application there is no code shared between the front end and backend
but if the project had a typescript or js frontend this would be as simple as creating a `shared` or
`common` package.

Lerna understands the dependencies between the packages and will build them in the correct order. We
also use its [changed](https://github.com/lerna/lerna/tree/master/commands/changed) functionality and 
some additional scripts to only deploy what has changed since a specified git ref. This could be the last
commit on master in the case of working on a branch or since the last merge in the case of merging a feature 
branch to master.

## Monitoring, Logging and Alerting
For logging and alerting we use [datadog](https://www.datadoghq.com/). As we are hosted in AWS we simply
give it access to our accounts via a specific cross account IAM role. Datadog is then able to ingest
all available cloudwatch metrics and we can then configure alerts from the metrics. These can be generic alerts e.g.
any lambda function that has reported an error or specific alerts like a lambda function has reached 80% of its
maximum allowed execution time.

For log aggregation we use Datadog logs. Logs are ingested via a datadog supplied lambda function
that forwards the cloudwatch logs written from our lambda functions. The log ingestor lambda function
ingests logs from api gateway and cloudfront too.

### Structured Logging
Datadog logs is one of many services available that works with structured logs. We use the [bunyan](https://github.com/trentm/node-bunyan) 
package to log messages and provide an object structure that serves as meta data for the log statement.

The following code shows how we would create new logger and log a message at the info level. We can then
define `jobId` as a facet in datadog and use it to find logs that are related to a specifc job.

```js
const log = bunyan.createLogger({name: "print-all"})

log.info({ jobId: job.id }, "Created a new job");
```

One downside to this approach is that we need to consider how to structure our logs upfront and ensure
that it is consistent across the whole application. In our case we started with something simple and
just tweaked it over time as we learn't what information was valuable when trying to troubleshoot
errors. Getting a application into the hands of users or at least testers early and then trying to understand
what is happening is a great way to learn how to improve your logging.

### LogRocket
[LogRocket](https://logrocket.com/) is a cool product that lets you cature and replay user sessions. This
is a great help when supporting your application. We have it intergrated with intercom and slack so that if a user
needs help they open intercom and chat with our support engineer who can then easily play back the user session
to see what the user was doing and see any errors that have occured.

## Lambda Layers
[Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html) are a relatively new feature
of AWS Lambda. They allow us to create a zip file of a common set of dependencies that do not change as
frequently as our application code. It also let us work around the size limits AWS places on the how large
our application bundle can be. We have created layers for:
- knexjs and associated mssql binaries
- libreoffice binary
- pdftk binary
- msgconvert (a bunch of perl scripts the convert .msg files to .eml files)

## Impact of Lambda Limits
As with most software platforms there are limitations. AWS Lamba has a set of [limits](https://docs.aws.amazon.com/lambda/latest/dg/limits.html)
that are well documented. We hit some of these limits along the way. I captured how we debugged some of them in another post
[Debugging Lambda Limits](https://www.sheakelly.com/blog/debugging-lambda-limits).

Here is how some of the limits impacted our application design:

### Storage
The heart of our solution is to convert many different types of individual files into an aggregated pdf that can
be easily downloaded and printed. Instead of downloading and printing 1000's of individual files and printing them
one by one we produce a one pdf file per quarter of the year e.g. 2010-Q1.pdf, 2010-Q2.pdf, 2010-Q3.pdf and so on.
Most of the time this is fine. However for clients that have a lot of progress notes and attachments breaking
them up into quarters results in files that are larger than we can store. The problem comes when we bundle pdfs
into quarters. Lambda restricts us to 512MB of storage under `/tmp`. As a results we only support up to 200MB pdf files. 
Otherwise we could run out of room in `/tmp`. We download up to 200MB of individual pdf file from S3 then run `pdftk` to merge them 
with a bit of room to spare.

### CPU and Memory
The limits around CPU and memory did not really impact our solution as we split up the processing of a file to run in an
individual lambda function execution. It rarely takes longer than few seconds to convert files to PDF.

### Function Duration
We were conscience of the fuction duration limit early on in the solution design. We implemented the processing so
that each lambda function did one thing quickly. When we create a new job i.e. `POST /job` we
return a new job id in the response and kick off another lambda asynchronously so as to not block the request and potentially
hit the 30 second timeout of API Gateway.

The conversion of files to PDF is structured so that the each function exection only converts one file to a PDF and 
them uploads the result to S3 and updates the status of processing in a dynamodb item. We are able to them process
a number of file concurrently to reduce the over conversion time.

### Process IDs
There is limit on number of processes that can run concurrently within the lambda container i.e. 1024. This should not
have been a problem as each lambda execution will simply run a command e.g. libreoffice and be done. Unfortunately we soon
discovered that libreoffice when run in headless mode can leak child processes that end up as `<defunct>` child
processes of the parent node js process. As the lambda container is reused for subsequent executions the defunct processes hang around and
we hit the 1024 process limit. We updated the libreoffice binaries which did improve things. Unfortunately the problem still
there is just took longer to happen.
After banging our heads against the wall for while we eventually decided to keep a count of the number of defunct
libreoffice processes. When it reaches a threshold of 800 we call `process.exit()` to terminate the nodejs process.
This releases the defunct processes. As we are processing items off a SQS queue the retry behaviour of lambda kicks
in and processing continues. Feels a little gross but it gets the job done.

## Dynamodb
For the storage of our Job, JobItem and Bundle entities we are using DyanmoDB. DynamoDB is a key value store. We can
easily provision a new table and stored our entities as json. DynamoDB is unlike a traditional relational store. It
took us a bit of time to understand how to used it successfully (and we had used it before). It is really
easy to simply store a single entity against a unique key e.g. a guid and retreive it by that key. Things get more complex
when you want to have relationships between items in the table. You need to plan your data access query patterns
and create the correct indexes. I could write a whole blog post on how we track the status of a job to determine
if it is complete or not. For now here are some great resources to learn how to use dyanmodb:
- [Advanced Design Patterns for DyanmoDB - Rick Houlihan](https://www.youtube.com/watch?v=HaEPXoXVf2k)
- [Best Practice for DyanmoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)

## Conclusions
So are we really killing trees with lambda functions? Well maybe. We are making it easier for our users to generate
large pdf files that get sent to the printer. As the process of producing the file is so much more efficient
it might happen more often than before. Hopefully in the future we can supply the pdf files directly on a usb key.
