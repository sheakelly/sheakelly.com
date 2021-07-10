---
slug: /blog/yow-2015-recap
title: "YOW! 2015 Recap"
description: Recap and thoughts on the the yow 2015 software developers conference
banner: YOW.png
date: 2015-12-21
---

This year (2015) I was lucky enough for the second year in a row to attend the YOW! software developers conference. This year the main themes were Micro service architectures, Docker and Go. There were elements of functional programming and some other really interesting items I will mention.
<!-- More -->

<img src="YOW.png" class="center-block img-responsive padded" >

# Day 1

## Adrian Cockcroft - It's complicated (Keynote)

Adrian is famous for his architectural work at Netflix during the companies transition to the cloud using Amazon Web Services (AWS). His keynote opened the conference and he covered a lot
of ground regarding complexity in software systems. He talked about how every day we do complex things like drive cars and take it for granted instinctively knowing how to operate them. There is a analogy
to Micro service architectures as they appear complex because we expose the complexity that was once hidden in a monolith. It will take us some time to get used to the complexity of Micro services and cloud
platforms but eventually we will be driving them like we drive cars today. He also showed off some of his research work around simulating IoT systems.

## Dan North - Delivery Mapping: Turning the Lights On

After working for 8 years at Thoughtworks helping to build the London office Dan is now an independent software and process consultant current working at scaling agile at a number of large American banks. The talk was about his insights into attempting to scale Agile in large organisations. Essentially he believes that the existing frameworks e.g. SAFe are not suited to this purpose. Scrum and XP are not suited to large multiple million dollar programs of work. He and one of his colleagues came to a similar conclusions about how to do this and are calling it 'Delivery Mapping'. Delivery Mapping attempts to find a high level range based estimate e.g. 3-9 months and then based on setting up a matrix of skills across the people in the organisation brings the right people to the work and then allows them to self organise around the problem. The skills matrix takes into account what people are good at and areas in which they want to grow. Dan seems to have a strong grasp of agile and is attempting to take the core ideas behind agile and is applying it at the macro level within organisations.

## Dave Hahn - A Day in the Life of a Netflix Engineer

Dave is a senior engineer at Netflix and covered a lot of the technical stack used at Netflix. He covered things like the 100s of micro services they have and that they have over 20000 EC2 instances deployed on Amazon's Web Services infrastructure. They are the classic unicorn all the horses aspire to being. Although Netflix host all their application in AWS one thing I learned is they design an maintain hardware known as OpenDirect. This is a specialised file server that is deployed at various ISPs across the world to deliver streaming content to their customers (of which I am one) as fast as possible. The design of this hardware is also open source. Impressive.

## Kathleen Fisher - Using Formal Methods to Eliminate Exploitable Bugs (Keynote)

An interesting talk on how software that is easily exploitable can be made more secure using [formal methods](https://en.wikipedia.org/wiki/Formal_methods). She covered the process of making a quad copter that could easily be compromised via its insecure WiFi connection and how it was harden using formal methods in conjunction with re architecting its design to isolate various components. There was a lot to this talk that was above my understanding but it was definitely interesting and an eye opener.

## Sam Newman - Deploying and Scaling Micro services

As Sam put it "Docker Docker Docker, Go Go Go, Micro services Micro services Micro services". There was a lot of Micro services content at YOW. This talk covered some interesting considerations around how to package your services and the impact on developers and operations people ranging from simple tarballs to docker containers. He ran through the various systems that can be used to docker containers. This included Docker Swarm, Apache Mesos and Google's Kubernettes. He had a very entertaining way of delivering this content. My biggest take away from this is that if your doing Micro services Docker is an good option for developers and operations people alike. Though the deployment story is a little immature. As we are using AWS we should trial the ECS service.

## Jame Lewis - Building Software that is Never Done

Another Thoughtworker Jame Lewis and Martin Fowler coined the term Micro services. He covered a lot of agile software engineering topics. He took time to explain that he believes that TDD is not dead and still suits most application development. It is more about using your experience to decide when and how rigorously it should be applied.

## Dave Thomas - Rigor Mortis (Avoiding)

Dave Thomas is one the Agile Manifesto guys. He is also a big Ruby guy and has spend that last 18 or so years writing Ruby development books for the Pragmatic series of books. He basically admitted that he had not challenged himself for a long time being happy with Ruby. However after discovering the relatively new Elixir language he has embraced functional programming and it has changed the way he approached software development. He went on to solve some interesting problems live coding style showing off pattern matching in Elixir. I was impressed. This talks made me want to further investigate functional programming.

## Amanda Launcher - Property Based Testing: Shrinking Risk In Your code

This was quite an interesting talk. However a fair bit of it went over my head. The essence of this talk was that tools like QuickCheck and the variants available for the various programming languages can be used along side traditional unit test techniques to improve code quality. Property based testing can be used to define how a function should behave. The testing tool then runs a number of generated test cases against your code adheres to the defined properties. This means we no longer have to hand write as many test cases and we should find more problems that we anticipated.

# Day 2

## Anita Sengupta and Kamal Oudrhiri - Engineering and Exploring the Red Planet (Keynote)

Anita and Kamal both work for NASA and were involved in the project that successfully landed Curiosity on Mars. A interesting talk on the engineering challenges they solved along the way. I was reminded that there is water flowing on Mars at various time of the year. They are trying to establish if there was any microbial life on mars. Talks like this really open your eyes to the complicated problems people are solving everyday. Also made me feel somewhat stupid :)

## Indu Alagarsamy - Sometimes the Questions are Complicated, but the Answers are Simple

Indu is a developer from Particular software, the company behind NServiceBus and supporting tools. Her talk evolved around gender bias and how it effects the IT industry. She had some interesting stories about growing up in India.

## Matt Ranney - Designing for Failure: Scaling Uber's Backend by Breaking Everything

Uber are another unicorn company that has had some interesting scaling problems. This talk went though some of their practices around dealing with failure in the Uber platform. Matt had a very entertaining story telling style. He told the story of how Uber's tech stack evolved from an outsourced php and mysql application to the its current micro services architecture. He also told the story about how Uber's largest outage was caused by their primary PostgreSQL database ran out of disk space and the troubles they had trying to get the system back. In total there was a 12 hour outage that could have been prevented if the on call engineers where aware or the significance of the alert messages they were receiving. The main take away from this talk was that we need to test failure scenarios frequently to ensure that the systems can adequately recover.

## Don Reinertsen - Thriving in a Stochastic World (Keynote)

This talk was an interesting look into product flow and how delivering incrementally and getting fast feedback is important in product development. It also highlighted that in the domain of software development there is no excuse for long slow release cycles. An interesting point made by Don was that if your product development cycle is too slow the market will change underneath you to the point of making your products out of date before release. Don made a lot of military references in his talk. One interesting point he made was the in the marines the platoon understands the objectives behind their mission so that can react appropriately if the situation changes from their initial orders. This is similar to the Sprint Goal concept in Scrum whereby the team should guide its decision making based on the overall goal of the sprint set by the Product Owner.

## Troy Hunt - Making Hacking Child’s Play

I have listened to a lot of podcast episodes feature Troy, so this talk was basically the best bits from all those podcast discussions. He showed off his pineapple WiFi man in the middle device. He launched a live DDOS attack to show just how easy it is for kids to launch them from their bedrooms.

## Alexandra Spillane and Matt Callanan - DevOps @ Wotif: Making Easy = Right

This talk covered Wotif's journey from infrequent releases that took their operation teams days to complete, to the adoption of continuously deployment from commits to master. Using their internal hardware they developed an alternative deployment process called SLIPway. SLIPway requires applications to adhere to an agreed set of requirements that are detected and reported by the release system. e.g. Application versioned, logging in a certain format, heath check endpoints available, automated deployment scripts in place etc. If the application didn't adhere to these it was rejected and forced back to the old release process. Wotif established a separate team consisting of developers and operations staff to define this process in consultation with the various development teams.

## Adrian Mouat - Using Docker Safely

Adrian just finished writing a book on Docker and in this talk went through some of the ways to secure docker images. These included using cut down linux distribution like alpine linux and running automated tools over containers. These represent a 'defence in depth' approach to security. He also showed how if you save your credentials in a Docker file system layer and then attempt to remove it all you need to do is to rollback subsequent layers and you can get at the credentials. I would like to get a copy of his book on running Docker to learn more in depth information about containers.

# Other Memorable Moments
As my organisation is looking to use SumoLogic for log aggregation in the cloud we were invited to attend the Beerops Sydney DevOps Christmas Meetup at the King Street Wharf Brewhouse (formerly the James Squire Brewhouse). With an open bar and plenty of craft beer on tap we had the opportunity to talk to a number of software vendors including section.io, IBM and Strut. I also got a cool SumoLogic stress ball and an AWS beer cooler.

At YOW itself there was a call out to congratulate Atlassian for a successful listing on the American Stock Exchange. Dave Thomas called out that "Simple solutions are not sexy" and "Beware CIOs who come with a new set of vendors" who bring a whole swag of complex and costly off-the-shelp products that are not developer friendly.

Then there was a "tech lead" form Tyro banking who when I ask my friend "What is Tyro?" turned around and gave me a 5 minute lecture about how great they were because they just got a banking license. Well done I guess?

I also got to hang out with 3 developers from my organisation, a number of DiUS consultants that I have had the chance to work with over the last 6 months, and I got to catch up with some guys who run the Newcastle Coder Group who I also used to work with at MinLog. Great bunch of people all round.
