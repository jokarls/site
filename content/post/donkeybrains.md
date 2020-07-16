+++
title = "Continious integration for Connect IQ apps with Docker and GitHub Actions"
date = 2020-07-06T16:47:26+02:00
tags = ["docker", "connect-iq", "garmin", "ci", "donkeybrains"]
author = "jokarls"
draft = false
+++

In 2015 when Garmin launched the Connect IQ platform, I was eager to try it out. Together with a friend, I started an ambitious app project and I knew that we need a build server of some sort. The solution back then was an old laptop running Jenkins, triggering builds on pushes to our GitHub repository.

Since then I've learned Docker and GitHub have released their Actions feature. So coming back to Connect IQ development, after a few years, those felt like good technologies to use when setting up a continuous integration pipeline.

## Creating a Connect IQ SDK Docker image

I decided to base the image on the Alpine Linux base image, to keep it as slim as possible. The Connect IQ SDK is based on Java so for the tools to work Java needs to be present in the image too.

You can find the source of the final image [here](https://github.com/jokarls/donkeybrains) and the image is can be found on Docker Hub [jokarls/donkeybrains](https://hub.docker.com/repository/docker/jokarls/donkeybrains)

## Setting up a GitHub CI Workflow