+++
title = "Continuous integration for Connect IQ apps with Docker and GitHub Workflows"
date = 2020-07-06T16:47:26+02:00
tags = ["docker", "connect iq", "garmin", "ci", "donkeybrains", "github"]
author = "jokarls"
draft = false
+++

In 2015 when Garmin launched the Connect IQ platform, I was eager to try it out. Together with a friend, I started an ambitious app project and I knew that we need a build server of some sort. The solution back then was an old laptop running Jenkins, triggering builds on pushes to our GitHub repository.

Since then I've learned Docker and GitHub have released their Actions feature. So coming back to Connect IQ development, after a few years, those felt like good technologies to use when setting up a continuous integration pipeline.

## Creating a Connect IQ SDK Docker image

I decided to base the image on the Alpine Linux base image, to keep it as slim as possible. The Connect IQ SDK is based on Java so for the tools to work Java needs to be present in the image too.

```Docker
FROM alpine:3.7 AS download

ARG VERSION=3.1.9-2020-06-24-1cc9d3a70

RUN apk --update add --no-cache curl unzip

WORKDIR /download
RUN curl -o connect-iq.zip https://developer.garmin.com/downloads/connect-iq/
    sdks/connectiq-sdk-lin-${VERSION}.zip && \
    unzip connect-iq.zip

FROM alpine:3.7

RUN apk --update add --no-cache openjdk8 bash

WORKDIR /connect-iq
COPY --from=download download/bin bin

ENV PATH $PATH:/connect-iq/bin

CMD ["monkeyc", "-v"]
```

You can find the source [here](https://github.com/jokarls/donkeybrains) and the final image is can be found on Docker Hub [jokarls/donkeybrains](https://hub.docker.com/repository/docker/jokarls/donkeybrains)

## Setting up a GitHub CI Workflow

GitHub workflows are yaml based and the one I created is pretty straight forward. It contains the four steps: **checkout**, **prepare key**, **build** and **store artifact**.

In the **prepare key** step I prepare the key that's used for signing the artifact in the build step. The key is base64 encoded and stored as a secret, so it doesn't have to be checked in with the source code.

The **build** step is where I instruct the workflow to use the donkeybrain image, along with the key from the previous step, to build the source.

In the final step, I use the predefined action *upload-artifact* to store the output from the build step. So that we can download it later.

```yml
name: CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    # Checkout source
    - name: Checkout
      uses: actions/checkout@v2

    # Prepare developer key from base64 encoded secret
    - name: Prepare key
      env:
        DEVELOPER_KEY: ${{ secrets.DEVELOPER_KEY }}
      run: echo $DEVELOPER_KEY | base64 -d > key.der

    # Build source using Donkeybrains docker image
    - name: Build
      uses: docker://jokarls/donkeybrains:latest
      with:
        args: monkeyc -d edge820 -o bin/app.prg -f monkey.jungle -y key.der

    # Store the artifact from previous step
    - name: Store artifact
      uses: actions/upload-artifact@v1
      with:
        name: app
        path: bin/app.prg
```

To put everything together, I created a small sample project: [donkeybrains-ci-sample](https://github.com/jokarls/donkeybrains-ci-sample)

And this is what it looks like, running on the simulator:

![running on simulator](/post/donkeybrains/simulator.png)

We now have a working CI pipeline for Connect IQ apps yay!


