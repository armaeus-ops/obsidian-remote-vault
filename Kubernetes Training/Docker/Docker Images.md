
Basic introduction to docker images, image creation, and handeling

## Step 1: Intro

In the previous exercise you pulled down images from the [Dockerhub registry](https://hub.docker.com) to run in your containers. Then you ran multiple instances and noted how each instance was isolated from the others. We hinted that this is used in many production IT environments every day but obviously we need a few more tools in our belt to get to the point where Docker can become a true time & money saver.

First thing you may want to do is figure out how to create our own images. While there are over 7,1 million images on the Dockerhub, it is almost certain that none of them are exactly what you run in your data center today. Even something as common as a Windows OS image would get its own tweaks before you actually run it in production. In the first lab, we created a file called `hello.txt` in one of our container instances. If that instance of our Alpine container was something we wanted to re-use in future containers and share with others, we would need to create a custom image that everyone could use.

We will start with the simplest form of image creation, in which we simply convert one of our container instances into an image using `commit`. Then we will explore a much more powerful and useful method for creating images: the `Dockerfile`.

We will then see how to get the details of an image through the inspection and explore the filesystem to have a better understanding of what happens under the hood.

Wait for Docker being installed:

Docker is installed via cloud-init. It may take some seconds for the installation to complete. Verify that the docker installation went through:

docker version

_Click to run on **node1**_

## Step 2: Image creation from a container

Let's start by running an interactive shell in a ubuntu container:

docker container run -ti ubuntu /bin/bash

_Click to run on **node1**_

As you know from earlier labs, you just grabbed the image called `ubuntu` from Docker Store and are now running the bash shell inside that container.

To customize things a little bit we will install a package called [figlet](http://www.figlet.org) in this container. Your container should still be running so type the following commands at your ubuntu container command line:

**Install figlet**

apt-get update
apt-get install -y figlet

_Click to run on **node1**_

**Call figlet**

figlet "hello docker"

_Click to run on **node1**_

You should see the words "hello docker" printed out in large ascii characters on the screen. Go ahead and exit from this container

exit

_Click to run on **node1**_

Now let us pretend this new figlet application is quite useful and you want to share it with the rest of your team. You _could_ tell them to do exactly what you did above and install figlet in to their own container, which is simple enough in this example. But if this was a real world application where you had just installed several packages and run through a number of configuration steps the process could get cumbersome and become quite error prone. Instead, it would be easier to create an _image_ you can share with your team.

To start, we need to get the ID of this container using the ls command (do not forget the -a option as the non running container are not returned by the ls command).

docker container ls -a

_Click to run on **node1**_

Before we create our own image, we might want to inspect all the changes we made. Try typing the command `docker container diff < container ID >` for the container you just created. You should see a list of all the files that were added to or changed in the container when you installed figlet. Docker keeps track of all of this information for us. This is part of the _layer_ concept we will explore in a few minutes.

Now, to create an image we need to "commit" this container. Commit creates an image locally on the system running the Docker engine. Run the following command, using the container ID you retrieved, in order to commit the container and create an image out of it.

docker container commit < CONTAINER_ID > 

That's it - you have created your first image! Once it has been commited, we can see the newly created image in the list of available images.

docker image ls

_Click to run on **node1**_

You should see something like this:

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              a104f9ae9c37        46 seconds ago      160MB
ubuntu              latest              14f60031763d        4 days ago          120MB

Note that the image we pulled down in the first step (ubuntu) is listed here along with our own custom image. Except our custom image has no information in the REPOSITORY or TAG columns, which would make it tough to identify exactly what was in this container if we wanted to share amongst multiple team members.

Adding this information to an image is known as _tagging_ an image.

TASK:

Tag the created image accordingly. From the previous command, get the ID of the newly created image and tag it so it's named `ourfiglet`:

docker image tag < IMAGE_ID > ourfiglet
docker image ls

Now we have the more friendly name `ourfiglet` that we can use to identify our image.

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ourfiglet           latest              a104f9ae9c37        5 minutes ago       160MB
ubuntu              latest              14f60031763d        4 days ago          120MB

Here is a graphical view of what we just completed: ![commit container to image](https://raw.githubusercontent.com/ebartz/play-with-docker.github.io/master/images/ops-images-commit.svg)

Now we will run a container based on the newly created `ourfiglet` image:

docker container run ourfiglet figlet hello

_Click to run on **node1**_

As the figlet package is present in our `ourfiglet` image, the command returns the following output:

 _          _ _
| |__   ___| | | ___
| '_ \ / _ \ | |/ _ \
| | | |  __/ | | (_) |
|_| |_|\___|_|_|\___/

This example shows that we can create a container, add all the libraries and binaries in it and then commit it in order to create an image. We can then use that image just as we would for images pulled down from the Docker Store. We still have a slight issue in that our image is only stored locally. To share the image we would want to _push_ the image to a registry somewhere. This is beyond the scope of this lab (and you should not enter any personal login information in these labs) but you can get a free Docker ID, run these labs, and push to the [Docker Community Hub](https://hub.docker.com) from your own system using Docker if you want to try this out.

As mentioned above, this approach of manually installing software in a container and then committing it to a custom image is just one way to create an image. It works fine and is quite common. However, there is a more powerful way to create images. In the following exercise we will see how images are created using a `Dockerfile`, which is a text file that contains all the instructions to build an image.

## Step 3: Image creation using a Dockerfile

Instead of creating a static binary image, we can use a file called a `Dockerfile` to create an image. The final result is essentially the same, but with a Dockerfile we are supplying the instructions for building the image, rather than just the raw binary files. This is useful because it becomes much easier to manage changes, especially as your images get bigger and more complex.

For example, if a new version of figlet is released we would either have to re-create our image from scratch, or run our image and upgrade the installed version of figlet. In contrast, a Dockerfile would include the `apt-get` commands we used to install figlet so that we - or anybody using the Dockerfile - could simply recompose the image using those instructions.

It is kind of like the old adage:

> _Give a sysadmin an image and their app will be up-to-date for a day, give a sysadmin a Dockerfile and their app will always be up-to-date_.

Ok, maybe that's a bit of a stretch but Dockerfiles are powerful because they allow us to manage _how_ an image is built, rather than just managing binaries. In practice, Dockerfiles can be managed the same way you might manage source code: they are simply text files so almost any version control system can be used to manage Dockerfiles over time.

We will use a simple example in this section and build a "hello world" application in Node.js. Do not be concerned if you are not familiar with Node.js: Docker (and this exercise) does not require you to know all these details.

We will start by creating a file in which we retrieve the hostname and display it.

INFO:

You should be at the Docker host's command line (`${vminfo:node1:hostname}:/#`). If you see a command line that looks similar to `root@b934ed40b559:/#` then you are probably still inside your ubuntu container from the previous exercise. Type `exit` to return to the host command line.

Restart SSH-Session:

If by any chance you have accidentally exited the SSH-Session by typing `exit` once too many you can just reload this webpage to create a new SSH-Session. You do not have to start the scenario once again.

Type the following content into a file named `index.js`. You can use vi, vim or several other Linux editors in this exercise. _ProTip: you can paste text from your clipboard into the shell with [Shift] + [Insert]_

index.js 

`var os = require("os"); var hostname = os.hostname(); console.log("hello from " + hostname);`

_Click to create index.js on **node1**_

The file we just created is the javascript code for our server. As you can probably guess, Node.js will simply print out a "hello" message. We will Docker-ize this application by creating a Dockerfile. We will use `alpine` as the base OS image, add a Node.js runtime and then copy our source code in to the container. We will also specify the default command to be run upon container creation.

Create a file named `Dockerfile` and copy the following content into it.

Dockerfile 

`FROM alpine RUN apk update && apk add nodejs COPY . /app WORKDIR /app CMD ["node","index.js"]`

_Click to create Dockerfile on **node1**_

Let's build our first image out of this Dockerfile and name it `hello:v0.1`:

docker image build -t hello:v0.1 .

_Click to run on **node1**_

How does docker know where my Dockerfile is?:

Did you noticed that you did not have to provide the Dockerfile to the `docker image build` command? The `docker image build` command searches for a file called `Dockerfile` in the working directory. If you want to provide a file called anything other than Dockerfile, or a Dockerfile in any other path, you can provide the `--file` or `-f` flag together with the path to your Dockerfile.

This is what you just completed: ![build container from dockerfile](https://raw.githubusercontent.com/ebartz/play-with-docker.github.io/master/images/ops-images-dockerfile.svg)

We then start a container to check that our applications runs correctly:

docker container run hello:v0.1

_Click to run on **node1**_

You should then have an output similar to the following one (the ID will be different though).

hello from 92d79b6de29f

**What just happened?** We created two files: our application code (`index.js`) is a simple bit of javascript code that prints out a message. And the `Dockerfile` is the instructions for Docker engine to create our custom container. This Dockerfile does the following:

1. `FROM <image>` specifies a base image to pull.
2. `RUN <cmd>`: executes commands inside that container which installs the Node.js server.
3. `COPY <from_host> <to_container>` copies a file (or folder) from the host to the container. The only file we have right now is our _index.js_ and the _Dockerfile_.
4. `WORKDIR <path>` specifies the directory the container should use when it starts up
5. `CMD <cmd>` gave our container a command to run when the container starts.

Recall that in previous labs we put commands like `echo "hello world"` on the command line. With a Dockerfile we can specify precise commands to run for everyone who uses this container. Other users do not have to build the container themselves once you push your container up to a repository (which we will cover later) or even know what commands are used. The _Dockerfile_ allows us to specify _how_ to build a container so that we can repeat those steps precisely everytime and we can specify _what_ the container should do when it runs. There are actually multiple methods for specifying the commands and accepting parameters a container will use, but for now it is enough to know that you have the tools to create some pretty powerful containers.

## Step 4: Image layers

There is something else interesting about the images we build with Docker. When running they appear to be a single OS and application. But the images themselves are actually built in _**layers**_. If you scroll back and look at the output from your `docker image build` command you will notice that there were 5 steps and each step had several tasks. You should see several "fetch" and "pull" tasks where Docker is grabbing various bits from Docker Store or other places. These bits were used to create one or more container _layers_. Layers are an important concept. To explore this, we will go through another set of exercises.

First, check out the image you created earlier by using the _history_ command (remember to use the `docker image ls` command from earlier exercises to find your image IDs):

docker image history < image ID >

What you see is the list of intermediate container images that were built along the way to creating your final Node.js app image. Some of these intermediate images will become _layers_ in your final container image. In the history command output, the original Alpine layers are at the bottom of the list and then each customization we added in our Dockerfile is its own step in the output. This is a powerful concept because it means that if we need to make a change to our application, it may only affect a single layer! To see this, we will modify our app a bit and create a new image.

Type the following in to your console window:

echo "console.log(\"this is v0.2\");" >> index.js

_Click to run on **node1**_

This will add a new line to the bottom of your _index.js_ file from earlier so your application will output one additional line of text. Now we will build a new image using our updated code. We will also tag our new image to mark it as a new version so that anybody consuming our images later can identify the correct version to use:

docker image build -t hello:v0.2 .

_Click to run on **node1**_

You should see output similar to this:

[+] Building 0.9s (9/9) FINISHED                                                                       docker:default
 => [internal] load build definition from Dockerfile                                                             0.0s
 => => transferring dockerfile: 131B                                                                             0.0s
 => [internal] load metadata for docker.io/library/alpine:latest                                                 0.7s
 => [internal] load .dockerignore                                                                                0.0s
 => => transferring context: 2B                                                                                  0.0s
 => [1/4] FROM docker.io/library/alpine:latest@sha256:c5b1261d6d3e43071626931fc004f70149baeba2c8ec672bd4f27761f  0.0s
 => [internal] load build context                                                                                0.0s
 => => transferring context: 1.77kB                                                                              0.0s
 => CACHED [2/4] RUN apk update && apk add nodejs                                                                0.0s
 => [3/4] COPY . /app                                                                                            0.0s
 => [4/4] WORKDIR /app                                                                                           0.0s
 => exporting to image                                                                                           0.0s
 => => exporting layers                                                                                          0.0s
 => => writing image sha256:177bb95f499f3451d8df35a13de8cf05eb9a39f509fc837ea92769a84520ddf8                     0.0s
 => => naming to docker.io/library/hello:v0.2                                                                    0.0s

Notice something interesting in the build steps this time. In the output it goes through the same five steps, but notice that in some steps it says **Using cache**.

![layers and cache](https://raw.githubusercontent.com/ebartz/play-with-docker.github.io/master/images/ops-images-cache.svg)

Docker recognized that we had already built some of these layers in our earlier image builds and since nothing had changed in those layers it could simply use a cached version of the layer, rather than pulling down code a second time and running those steps. Docker's layer management is very useful to IT teams when patching systems, updating or upgrading to the latest version of code, or making configuration changes to applications. Docker is intelligent enough to build the container in the most efficient way possible, as opposed to repeatedly building an image from the ground up each and every time.

## Step 5: Image Inspection

Now let us reverse our thinking a bit. What if we get a container from Docker Store or another registry and want to know a bit about what is inside the container we are consuming? Docker has an `docker inspect` command for images and it returns details on the container image, the commands it runs, the OS and more.

The `alpine` image should already be present locally from the exercises above (use `docker image ls` to confirm), if it's not, run the following command to pull it down:

docker image pull alpine

_Click to run on **node1**_

Once we are sure it is there let's inspect it.

docker image inspect alpine

_Click to run on **node1**_

There is a lot of information in there:

- the layers the image is composed of
- the driver used to store the layers
- the architecture / OS it has been created for
- metadata of the image
- ...

We will not go into all the details here but we can use some filters to just inspect particular details about the image. You may have noticed that the image information is in JSON format. We can take advantage of that to use the inspect command with some filtering info to just get specific data from the image.

Let's get the list of layers:

docker image inspect --format "{{ json .RootFS.Layers }}" alpine

_Click to run on **node1**_

Alpine is just a small base OS image so there's just one layer:

["sha256:60ab55d3379d47c1ba6b6225d59d10e1f52096ee9d5c816e42c635ccc57a5a2b"]

Now let's look at our custom Hello image. You will need the image ID (use `docker image ls` if you need to look it up):

docker image inspect --format "{{ json .RootFS.Layers }}" < image ID > 

Our Hello image is a bit more interesting (your sha256 hashes will vary):

["sha256:5bef08742407efd622d243692b79ba0055383bbce12900324f75e56f589aedb0",
"sha256:5ac283aaea742f843c869d28bbeaf5000c08685b5f7ba01431094a207b8a1df9",
"sha256:2ecb254be0603a2c76880be45a5c2b028f6208714aec770d49c9eff4cbc3cf25"]

We have three layers in our application. Recall that we had the base Alpine image (the `FROM` command in our Dockerfile), then we had a `RUN` command to install some packages, then we had a `COPY` command to add in our javascript code. Those are our layers! If you look closely, you can even see that both _alpine_ and _hello_ are using the same base layer, which we know because they have the same sha256 hash.

The tools and commands we explored in this lab are just the beginning. Docker Enterprise Edition includes private Trusted Registries with Security Scanning and Image Signing capabilities so you can further inspect and authenticate your images. In addition, there are policy controls to specify which users have access to various images, who can push and pull images, and much more.

Another important note about layers: each layer is immutable. As an image is created and successive layers are added, the new layers keep track of the changes from the layer below. When you start the container running there is an additional layer used to keep track of any changes that occur as the application runs (like the "hello.txt" file we created in the earlier exercises). This design principle is important for both security and data management. If someone mistakenly or maliciously changes something in a running container, you can very easily revert back to its original state because the base layers cannot be changed. Or you can simply start a new container instance which will start fresh from your pristine image. And applications that create and store data (databases, for example) can store their data in a special kind of Docker object called a _**volume**_, so that data can persist and be shared with other containers. We will explore volumes in a later lab.

Up next, we will look at more sophisticated applications that run across several containers and use Docker Compose and Docker Swarm to define our architecture and manage it.

## Step 6: Image Scanning

### Why is Image Scanning Important in Docker Image Creation?

When building Docker images, especially in a DevOps context where speed and efficiency are crucial, security can often become an afterthought. However, integrating security into the image creation process is vital to protect against vulnerabilities that could be exploited in production. Here’s why image scanning is essential:

### **Early Detection of Vulnerabilities**

Scanning images for vulnerabilities during the build process allows teams to detect and fix security issues before the images are used in production environments. This preemptive action drastically reduces the risk of deploying containers with exploitable weaknesses.

### **Compliance and Governance**

Many industries require compliance with specific security standards (e.g., PCI DSS, HIPAA, GDPR). Regular scanning of Docker images helps ensure that the containers meet these compliance requirements throughout their lifecycle, avoiding hefty fines and legal issues.

### **Integration into CI/CD Pipelines**

Image scanning tools like Trivy can be seamlessly integrated into CI/CD pipelines, which helps automate the security checks and ensures every image is scanned before deployment. This automation supports a DevSecOps approach, promoting security as a shared responsibility among all stakeholders involved in the development process.

### **Cost-effective Security**

Identifying and addressing security vulnerabilities early in the development process is significantly less costly than dealing with a security breach after deployment. Early detection saves resources and protects the organization's reputation by preventing potential security incidents.

### **Building Trust**

Regularly scanned and secured images build trust with customers and users by ensuring that the applications run in a secure and stable environment. It demonstrates a commitment to security and reliability.

Incorporating image scanning into Docker image creation not only enhances security but also supports regulatory compliance, reduces potential costs associated with late-stage fixes, and helps maintain customer trust. For best practices, tools like Trivy are recommended due to their ease of use, comprehensive vulnerability detection, and integration capabilities with existing CI/CD pipelines.

For more on integrating image scanning in your image building process, check out the [Trivy documentation](https://aquasecurity.github.io/trivy/v0.29.2/).

### Introduction to Trivy

[Trivy](https://github.com/aquasecurity/trivy) is a comprehensive, open-source security scanner that simplifies the process of finding vulnerabilities in container images, file systems, and even Git repositories. Developed by [Aqua Security](https://www.aquasec.com/), Trivy is easy to install and integrates seamlessly into CI/CD pipelines for automated security checks. Its main features include:

#### Key Features:

- **Wide Range of Scanning Targets**: Trivy can scan container images, local file systems, and source code repositories.
- **Comprehensive Vulnerability Detection**: It detects vulnerabilities in OS packages (Alpine, Red Hat, etc.) and major programming language dependencies.
- **Easy Integration**: Trivy can be used standalone, run as a client-server, or integrated into CI systems like Jenkins, GitLab CI, and GitHub Actions.
- **High Accuracy**: Includes a comprehensive database of vulnerabilities that is regularly updated to minimize false positives.

#### Usage Scenarios:

- **CI/CD Integration**: Automate scanning during builds to ensure vulnerabilities are caught before deployment.
- **Pre-Deployment Checks**: Run as a part of deployment scripts to scan environments and configurations.
- **DevSecOps**: Incorporate into security practices to ensure continuous security monitoring.

For detailed usage and integration guides, visit the [Trivy GitHub page](https://github.com/aquasecurity/trivy).

### Install Trivy

To get trivy installed we need to add the needed repositories and use our package manager do perform the installation.

apt-get install -y wget apt-transport-https gnupg lsb-release jq
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
apt-get update

_Click to run on **node1**_

apt-get install -y trivy

_Click to run on **node1**_

### Using Trivy

Trivy is a versatile tool that can scan container images, file systems, and even repositories for security vulnerabilities. Here's how to get started with scanning Docker images and other basic operations:

trivy image hello:v0.1

_Click to run on **node1**_

Trivy can output results to a file in several formats such as table, JSON, and custom HTML reports.

This is an example of the json output:

trivy image -f json -o results.json hello:v0.1
cat results.json | jq

_Click to run on **node1**_

In some cases it makes sense to filter the results for `HIGH` and `CRITICAL` vulnerabilities.

trivy image --severity HIGH,CRITICAL svahub.io/library/wordpress:latest

_Click to run on **node1**_

To include trivy in cour CI/CD pipelines, you could simply add this command:

trivy image --exit-code 1 --severity CRITICAL hello:v0.1

_Click to run on **node1**_

## Step 7: Terminology

- **Layers** - A Docker image is built up from a series of layers. Each layer represents an instruction in the image's Dockerfile. Each layer except the last one is read-only.
- **Dockerfile** - A text file that contains all the commands, in order, needed to build a given image. The [Dockerfile reference](https://docs.docker.com/engine/reference/builder) page lists the various commands and format details for Dockerfiles.
- **Volumes** - A special Docker container layer that allows data to persist and be shared separately from the container itself. Think of volumes as a way to abstract and manage your persistent data separately from the application itself.