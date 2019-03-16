---
title: Configuring Travis CI for Github builds
layout: post
date: '2019-03-16 01:18:06 +0530'
categories: github CI dev automated-builds
---

Travis CI is a Continous Integration Tool that is widely used by many developers around the world to build, deploy and release their code that is hosted in GitHub repositories. My previous post on Jenkins also explored the possibilty of CI which it is perfectly capable of doing. Travis CI is much more simpler and easier to setup and run builds than Jenkins. So this post is aimed more at newbies (including myself 😉).

When Travis CI is integrated with GitHub, it automatically detects changes to the repositories via webhooks and builds and/or run tests for that codebase. How Travis CI knows how to build or test codebases is defined in ``travis.yml`` file in the project root.

This post will be a simple introduction into configuring continous integration into a small GitHub repository. So, let's get started.
