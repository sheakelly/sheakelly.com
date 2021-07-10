---
slug: /blog/thoughts-on-mono-repos
title: "Mono Repositories"
description: Thoughts on Mono Repositories
quote: Mono repositories allow us to reduce the effort of maintaining many indivdual repositories
banner: monorail.jpg
date: 2018-09-24
---

# What is a mono repo

As the name suggests a mono repository has _all_ the code for a solution space in a single source code repository. I use the term "solution space" rather than product or project because there different types of mono repositories. The most extreme examples are at Google and Facebook where they apparently store most of the organisations code in a single repository. Google have published a [paper](https://cacm.acm.org/magazines/2016/7/204032-why-google-stores-billions-of-lines-of-code-in-a-single-repository/fulltext) on their implementation and presented on the topic in [Why Google Stores Billions of Lines of Code in a Single Repository](https://www.youtube.com/watch?v=W71BTkUbdqE)

## Monolithic services

Having one source repository does not mean we have to have one monolithic deployable unit. A mono repo will often house many independently deployable services and front end applications. It is therefore compatible with both micro service and majestic monolith designs.

# So many packages

The rise of software package managers has made it really easy to add external libraries as dependencies to our software projects. This coupled with the rise of newer runtimes like nodejs in the age of opensource has caused the proliferation of tiny libraries. Javascript projects can quickly depend on 1000's of external libraries. Having lots of single purpose packages is great as a consumer of the packages especially when the packages are stable and do not change very often. This approach can however problematic when working on a larger product or project. Usually as the project or product grows in size and complexity the tendency is to break it up into various reusable modules or packages. When these are stored in separate repositories we need to update not only the package itself but also update the package across all projects that depend on it. In the mono repo the update is applied to all dependent package by default and we get feedback from the compiler or test suites about the impact of the change right away. This does mean more upfront work it the change break an api contract however I think this is an ok trade off. The alternative is having to manage many version of a single package. Techniques like semantic versioning can help here but there is a slower feedback loop if there are any incompatibilities.

# The Old Days

For those of us that predate the javascript era of software development "everything old is new again". Thinking back to my work using .Net and before that Java most repositories where a mono repository for a project. One .Net solution or one Maven project with many modules. The move from lots of tiny packages to mono repo style structures shows a level of maturity in the Javascript community. Javascript has grown up and we now have choice about how to structure our software projects which is a good thing.

# Advantages

## Less repositories

Less things to manage, configure and control access to. It also allows us to see the structure of a solution in one place in a single directory structure.

## Shared dependencies

Often in a mono repo development dependencies are _hoisted_ to the top level project thus all packages within the mono repo will share the same version of common dependencies.

## Less dev setup

When working with lots of separate repositories we need to configure language and linting rules for every repo. When using tools like typescript there are often shared `tsconfig.json`, `tslint.json` files and \*.d.ts declaration files that are either duplicated or bundled up in yet another package. This is also the case for straight es(5|6|7) javascript projects using babel and eslint.

In a mono repo we can have one set of compiler settings, linting and code formatting rules that are applied uniformly. If we change a rule we can see the impact and the code is consistent. This also promotes more collaboration around changes in this space.

# Disadvantages

## Large checkouts

A local copy of a mono repo may be quite large if there are 1000's of files and a long commit history. At large organisations it could quite easily be larger that your local disk can hold. There are ways around this e.g. retrieving only the latest revision of each file. Google even created their own source control system to deal with this issue.

## Deployment complications

As the number of deployable projects grows it becomes impractical to deploy everything in the repo on every commit. Some mechanism for detecting and deploying only those project that have changed is required. Unfortunately at the time of writing most CI/CD systems are not able to do this out of the box and some custom scripting will be needed.

# Javascript Mono Repositories

![lernajs](lernajs.jpg)
Lerna is an opensource project from the babeljs team that makes it easier to manage having multiple nodejs packages in the one repository. Examples of projects doing this include babel (who created lerna), react and gatsbyjs. A larger list exists on the lerna project site.

# So what should I do?

As always it depends. Personally I prefer to start with a single repository for a project or product only splitting modules out if they are fairly stable and not directly related to a specific project of product. Something that _could_ be open sourced with distinct value outside of a specific organisation is a candidate for a new repository. I have not worked in an organisation that has only one repo for everything. Even google have separate repositories for angular and chrome.

I think less is more when it comes to setting up build scripts and development dependencies. Also having all the code you need to change for a new feature in one repository mean less messing around with pre-release versions of dependencies and make pull requests easier to understands.

In languages that have stronger type systems having all the code in the same repo makes it easier to detect which compiled dependencies are affected by interface or contract changes.

For javascript projects I think it is important to not take a default stance and blindly create hundreds of small repositories. It is worth considering the alternative.

# Links

Here are the links I used writing this post:

* https://cacm.acm.org/magazines/2016/7/204032-why-google-stores-billions-of-lines-of-code-in-a-single-repository/fulltext
* https://www.youtube.com/watch?v=W71BTkUbdqE
* https://lernajs.io/
* https://trunkbaseddevelopment.com/monorepos/
* https://medium.com/@maoberlehner/monorepos-in-the-wild-33c6eb246cb9
* https://github.com/korfuri/awesome-monorepo
