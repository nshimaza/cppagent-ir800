# MTConnect Agent on IR800 IOx

This Git repo provides procedure and Dockerfiles to build an IOx
application running open source C++ MTConnect Agent on
Cisco 829 / 809 Industrial Integrated Services Routers.

Building process consists of following steps.

1. Build MTConnect Agent executable using Cisco provided Docker image.
1. Extract the built executable from the image.
1. Build Docker image for runtime.
1. Convert the runtime image to IOx application.

## Prepare your environment

Before you start, you need to perform two steps bellow.

* Install `ioxclient`
* Let Docker login to [devhub.cisco.com](devhub.cisco.com).

`ioxclient` is a developer tool used for manipulating IOx environment.
It will be used for creating IOx application from a Docker image.

Cisco provides Docker image tailored for IOx application.  It can be used both
for build environment and runtime environment.  The image is distributed via [devhub.cisco.com](devhub.cisco.com).  You need to let your
Docker logged in to the side with following command so that you can obtain
the image during `docker build` process.

```shell-session
$ docker login -u iox_docker.gen -p AKCp2WX2yZXhwiQXG2xZjHvnrov8HA2HkXZyH3Z46ErGNMDz1fHQKAMRCBbEvseLowd7BQ8S1 devhub-docker.cisco.com
```

Consult to [Cisco DevNet site](https://developer.cisco.com/site/iox/docs/#docker-images-and-packages-repository) for more detail.

## Build executable in Docker image

Build MTConnect Agent executable using Docker image from Cisco.
The Dockerfile under build directory does it for you.
Execute following command.

```shell-session
$ cd build
$ docker build -t cppagent-ir800-build .
```

You will get a Docker image tagged with "cpp-agent-ir800-build".

## Extract executable from the image

The executable is built at /opt/src/cppagent/agent/agent.
You can extract it by executing following command.

```shell-session
$ cd ../runtime
$ docker run --rm -ti --volume "`pwd`:/vol" cppagent-ir800-build /bin/sh /copy.sh
```

You will get x64 executable file named "agent" on your working directory.

## Build runtime Docker image

Make sure you already have a x64 executable file named "agent" under runtime directory.

The Dockerfile under the runtime directory builds a Docker image which
contains the "agent" executable and necessary dynamic libraries for you.
To get the image, execute following command under runtime directory.

```shell-session
$ docker build -t cppagent-ir800 .
```

You will get a Docker image tagged with "cppagent-ir800".
The image contains agent executable and start-up shell script.

## Convert Docker image to IOx app

Once a Docker image for runtime has been built, you can convert it to
IOx application package.

Navigate to the directory `app` and execute following command.

```shell-session
$ cd ../app
$ ioxclient docker package cppagent-ir800 .
```

You will get `packaget.tar` which is IOx application package installable to IR809 or IR829.
