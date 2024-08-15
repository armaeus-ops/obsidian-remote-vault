Lets try to solve some issues

## Step 1: Intro

In this Lab you will need to take a deeper look at some Dockerfiles to make improvements.

The developers worked on those images with best intends but it seems like there is still room for improvement.  
  

Remember that there are useful tips also in the [documentation of Docker](https://docs.docker.com/). Especially these two articles outline good practices for Images and also Dockerfiles:

- [Overview of best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Image-building best practices](https://docs.docker.com/get-started/09_image_best/)

  
  

![Broken Docker](https://images.idgesg.net/images/article/2021/08/sinking-cargo-ship_docker_storage_colorful-shipping-containers_by-tom.ruethai_shutterstock_202572958-100898013-large.jpg?auto=webp&quality=85,70)

## Step 2: Preperation

Clone Source

Before we can start we need to get some files and install a few packages.

git clone https://github.com/Cristoforo86/docker /tmp/docker
cp -r /tmp/docker/* .
apt -y install tree jq

_Click to run on **node1**_

Now you should be able to see some files in your homedir. You can get an overview with `tree`

tree .

_Click to run on **node1**_

This should look like this:

.
├── bad-practice
│   ├── app.js
│   ├── Dockerfile
│   ├── package.json
│   └── task.txt
├── react-multi
│   ├── Dockerfile
│   ├── package.json
│   ├── package-lock.json
│   ├── public
│   │   ├── favicon.ico
│   │   ├── index.html
│   │   ├── logo192.png
│   │   ├── logo512.png
│   │   ├── manifest.json
│   │   └── robots.txt
│   ├── README.md
│   └── src
│       ├── App.css
│       ├── App.js
│       ├── App.test.js
│       ├── index.css
│       ├── index.js
│       ├── logo.svg
│       ├── reportWebVitals.js
│       └── setupTests.js
└── README.md

4 directories, 23 files

Now lets get started!

## Step 3: Bad Practice Image

Improve the present Dockerfile by applying good practice principles. This Dockerfile should be edited regarding security concerns, image build time and efficiency.

  
  

For this task change the directory to `~/bad-practice`

cd ~/bad-practice

_Click to run on **node1**_

Now try to improve the image build by applying the good practices regarding images and Dockerfiles.

Hints

- there is no need to install certain packages if the right base image is chosen
- which user should run in the container?
- should all files be copied into the image?

## Step 4: Multi Stage

### Tasks

Create a Multi Stage Dockerfile for a React App. You will need two stages: a build stage where you build the React App and its static content and a second stage which is needed to serve the website with a nginx web server.

  
  

For this task change the directory to `~/react-multi`

cd ~/react-multi

_Click to run on **node1**_ Hints

- Copy the package.json and package-lock.json (if exists) into the image
- to build the static content (compiled CSS and JavaScript) you will need to run "npm run build"
- the built static content (look at ./build) needs to be copied into the second stage, so the nginx web server can serve it
- the nginx server is started by running "nginx -g daemon off"
- Add content to the .dockerignore file as you do not want to copy every file into the image