---
title: Configuring Travis CI for Github builds
layout: post
date: '2019-03-16 01:18:06 +0530'
categories: github CI dev automated-builds
---

Travis CI is a Continous Integration Tool that is widely used by many developers around the world to build, deploy and release their code that is hosted in GitHub repositories. My previous post on Jenkins also explored the possibilty of CI which it is perfectly capable of doing. Travis CI is much more simpler and easier to setup and run builds than Jenkins. So this post is aimed more at newbies (including myself 😉).

## Introduction

When Travis CI is integrated with GitHub, it automatically detects changes to the repositories via [Webhooks](https://developer.github.com/webhooks/) and builds and/or run tests for that codebase. How Travis CI knows how to build or test codebases is defined in ``.travis.yml`` file in the project root.

This post will be a simple introduction into configuring continous integration into a small GitHub repository. So, let's get started.

## Steps, Follow ‘em 👌 

Head on to [Travis CI](https://travis-ci.org ) and signup using your GitHib account. Travis CI will then prompt for authorization on your repositories.

![Signup](/assets/travis-ci/signup.PNG "Signup on Travis CI")

Once you’re successfully signed in, Travis CI will then import your repository details. This might take a while depending on your repository count. After the import is complete, you will have a list of repositories as shown below.

![Repo list](/assets/travis-ci/repos.PNG "Repositories")

For this demostration, I’m using my [weather-app](https://github.com/amodsachintha/react-weather-app ) created with React. Click on **“Activate Repository”** to enable/add a [Webhook](https://developer.github.com/webhooks/) to the repository. This Webhook will trigger configured services for certain events.

![Weather-App](/assets/travis-ci/weather-app.PNG "React Weather App")

So, like I said before, Travis CI will not know what to do as soon as you Activate the repository. We will now configure the ``travis.yml`` file in the GitHub repository for Travis CI to get the required build and deploy configuration.

![travis.yml](/assets/travis-ci/travis.yml.PNG "Travis CI config")

Note that I’m using [Surge.sh](https://surge.sh) to deploy my static build to Surge under [amodsachintha-react-weather.surge.sh](http://amodsachintha-react-weather.surge.sh ). Therefore, we need to go back to Travis and set up a couple things. Surge will need your email address and a token to log in as you from the TravisCI build tool. To get this token, run `surge token` in your command line and copy the output from the second line, I masked mine, but it should look something like this: 

![Surge Token](/assets/travis-ci/surge-token.PNG "Surge token")

Now, click on the **More Options** dropdown and select settings. Under the Environment Variables section add the surge token and login email as follows.

![Surge ENV](/assets/travis-ci/surge-env.PNG "Surge environment variables")

Okay now, we’re almost done. The final step is to commit and push the changes to the GitHub remote that were made to the weather-app repository.

![git commit and push](/assets/travis-ci/github-desktop.PNG "git commit and push")

Head on to Travis CI to watch the build log scrolling...

![Build start](/assets/travis-ci/travis-ci-build-start.PNG "Build start")

After the build is complete, inspect the log to see if the build and deploy was indeed sucessful.

![Build complete](/assets/travis-ci/travis-ci-build-complete.PNG "Build complete")

Additionally, we can add a fancy build badge to our README.md on GitHub for users to see if the builds are successful 😎 .  Click on the build bage near the repository name on Travis CI and select the Markdown version of the build status image. We can then add this code to the README file. Nice stuff!

![Get Build Badge](/assets/travis-ci/build-badge.PNG "Get Build Badge")

It’ll look just fine.

![Get Build Badgel](/assets/travis-ci/build-badge-github.PNG "Get Build Badge GitHub")

<br><br>

## Next Steps

![Deployed App](/assets/travis-ci/deployed-app.png "Deployed App")

There’s a lot more you can do with these tools. I hope this was an informative look at some of the current web development tools and how to leverage them to suit your development needs. So long 👋 .