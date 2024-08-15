## Step 1: Intro

Docker Images should be as small as possible. They should onl contain what is needed to run your application.

During the setup process it may be needed to install different build tools to actually build the software. But not everything thats needed to build software is required for running the software.

Whenever this is the case, we can use multi staged build to seperate the build process from the image that just runs the application.

In this scenario we will:

- learn how to use multi staged builds
- use docker best practices
- reduce image size using multi staged builds

## Step 2: The old way

A common pipe-line for building applications in Docker involves adding SDKs and runtimes, followed by adding code and building it. The most efficient way to get a small image tends to be to use 2-3 Dockerfiles with different filenames where each one takes the output of the last. This is referred to as the `Builder pattern` in the Docker community.

This lab explores a feature called Multi-stage builds.

Let's build a simple Golang application which counts internal/external facing anchor tags to help us come up with an SEO rating.

Let's try out the href-counter Docker image from the hub, then look at how to re-build from the Github repository:

docker run -e url=https://www.sva.de/de alexellis2/href-counter

_Click to run on **node1**_

{"internal":258,"external":6}

You get a JSON object returned giving the total amount of internal vs external links.

Let's clone the source:

git clone https://github.com/alexellis/href-counter
cd href-counter

_Click to run on **node1**_

Cloning into 'href-counter'...
remote: Counting objects: 24, done.
remote: Compressing objects: 100% (19/19), done.
remote: Total 24 (delta 7), reused 12 (delta 1), pack-reused 0
Unpacking objects: 100% (24/24), done.

## Step 3: The old way of doing things

Let's build the Docker image with all the Golang toolchain and see how big the image comes out as:

docker build -t alexellis2/href-counter:sdk . -f Dockerfile.build

_Click to run on **node1**_

Now check the size of the image:

docker images |grep href-counter | grep sdk

_Click to run on **node1**_

docker images |grep href-counter
alexellis2/href-counter   sdk       d531cef695cd   57 seconds ago   839MB

The `docker history` command will show you that the layers we added during the build are only a small part of the resulting image (about 20MB +/-):

docker history alexellis2/href-counter:sdk |head -n 4

_Click to run on **node1**_

IMAGE               CREATED             CREATED BY
                   SIZE                COMMENT
f8b1953fb9c7        1 second ago        /bin/sh -c CGO_ENABLED=0 GOOS=linux go bui...   5.64MB
5d24895500e8        9 seconds ago       /bin/sh -c #(nop) COPY file:d3eec1f1fefbec...   1.71kB
d83dc0785057        9 seconds ago       /bin/sh -c go get -d -v golang.org/x/net/html   13.6MB
c6f59b210906        11 seconds ago      /bin/sh -c #(nop) WORKDIR /go/src/github.c...   0B

The image is quite large, but this Golang package can be built into a very small binary with no external dependencies, then added to an Alpine Linux base image.

## Step 4: The builder pattern

Lets have a look at the `build-multi-dockerfiles.sh` so you can see how the builder pattern uses two separate steps that would usually happen in different Dockerfiles.

shell 

`#!/bin/sh echo Building alexellis2/href-counter:build  docker build -t alexellis2/href-counter:build . -f Dockerfile.build  docker create --name extract alexellis2/href-counter:build  docker cp extract:/go/src/github.com/alexellis/href-counter/app ./app docker rm -f extract  echo Building alexellis2/href-counter:latest  docker build --no-cache -t alexellis2/href-counter:latest .`

./build-multi-dockerfiles.sh

_Click to run on **node1**_

As you can see there are quite a few intermediate steps required to create an optimized image using the _Builder pattern_.

Let's see how big the Docker image came out as:

docker images |grep alexellis2/href-counter | grep latest

_Click to run on **node1**_

This is much smaller than when we built our first image with the Golang SDK included.

## Step 5: Multi-stage build example

While the builder pattern helps us achieve a small image, it does require extra work for every piece of software we want to package in Docker.

Here is where multi-stage builds help us out. Instead of using a shell script to orchestrate two separate Dockerfiles, we can just use one and define stages throughout.

To use the Dockerfile.multi file in the Github repository to do a multi-stage build, then check the size of the resulting image against that of the image created by the Golang SDK base and the builder pattern.

docker build -t alexellis2/href-counter:multi . -f Dockerfile.multi

_Click to run on **node1**_

docker images | grep multi

_Click to run on **node1**_

alexellis2/href-counter   multi     46160507bb7e   11 seconds ago   13.4MB

The image now contains just what we need to actually run the application and we were able to reduce the image size by 98%.

## Step 6: Using FROM scratch

In case our application is just an executable binary that does not rely on any OS binaries, we can reduce the size of the base image even more.

The images `scratch` is used if you just want to put executable binaries inside of a container.

Here is an example of building "Hello World" in Go.

Create a new folder:

cd
mkdir hello
cd hello

_Click to run on **node1**_

Create the app.go file:

echo 'package main

import "fmt"

func main() {
    fmt.Println("Hello world!")
}
' | tee app.go

_Click to run on **node1**_

Create a Dockerfile with the following contents:

echo '
FROM golang:1.7.3
COPY app.go .
RUN go build -o app app.go

FROM scratch
COPY --from=0 /go/app .
CMD ["./app"]
' | tee Dockerfile

_Click to run on **node1**_

Now build and run the Dockerfile:

docker build -t hello-world-lab .
docker run hello-world-lab

_Click to run on **node1**_

The resulting size of hello-world is very small:

docker images |grep hello-world-lab

_Click to run on **node1**_

## Step 7: nodejs example

Now that we had multiple go examples, lets try the same with Nodejs.

cd
mkdir nodejs
cd nodejs

_Click to run on **node1**_

Let’s create a simple node application:

echo 'console.log("Hello, pkg")' | tee index.js

_Click to run on **node1**_

We’ll now package it using a very basic dockerfile

echo 'FROM node:16.8
COPY . /app
CMD ["node", "/app/index.js"] ' | tee Dockerfile

_Click to run on **node1**_

Finally, we’ll build our image and check it’s size:

docker build -t myapp .
docker image ls | grep myapp

_Click to run on **node1**_

You can see that our `myapp` is 900+MB

## Step 8: Using PKG and multi-stage builds

Let’s create now a new Dockerfile that bundles our app with pkg and uses multi-stage builds to package it in a smaller docker image

echo 'FROM node:16.8
RUN npm install -g pkg pkg-fetch

ENV NODE node16
ENV PLATFORM alpine
ENV ARCH x64

RUN /usr/local/bin/pkg-fetch ${NODE} ${PLATFORM} ${ARCH}

COPY . /app
WORKDIR /app
RUN /usr/local/bin/pkg --targets ${NODE}-${PLATFORM}-${ARCH} index.js

FROM alpine
COPY --from=0  /app/index /
CMD ["/index"]' | tee Dockerfile-pkg

_Click to run on **node1**_

We’ll build our app the same way as before but using our new Dockerfile:

docker build -t myapp_pkg -f Dockerfile-pkg .
docker image ls | grep myapp_pkg

_Click to run on **node1**_

As you can see, our new docker image is around 50MB and it works as expected:

docker run myapp_pkg

_Click to run on **node1**_

## Step 9: Conclusion

Multi stage builds have the potential to reduce the image size dramatically.

But that is not the only reason why it makes sense to strip down containers to their minimum.

Whenever you ship containers that contain all the dependencies which are needed to build software, that also means that a potential attacker has all the tools needed to build things. Also when it comes to patching it is more likely that you have to fix securtiy issues in lager images containing OS dependencies that you would have in smaller images that just contain the bare minumum.