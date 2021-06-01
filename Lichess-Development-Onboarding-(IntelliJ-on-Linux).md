This page shows you how to set up `lila` on an [Ubuntu Linux](https://releases.ubuntu.com/20.04/) machine with [IntelliJ IDEA](https://www.jetbrains.com/idea/).

With this setup as a newcomer to [Scala](https://www.scala-lang.org/) and [SBT](https://www.scala-sbt.org/), you will greatly benefit from powerful features like autocompletion, library dependency management, code formatting, debugging functionalities and many more things.

# Prerequisites
* [Ubuntu Linux](https://releases.ubuntu.com/20.04/)
* [IntelliJ IDEA](https://www.jetbrains.com/idea/)
* [git](https://git-scm.com/)
* [JDK](https://openjdk.java.net/install/)

Optionally (for an easy `redis` and `mongodb` setup): [docker-compose](https://docs.docker.com/compose/install/)

# Instructions 

## Scala plugin

Install the official [Scala plugin](https://plugins.jetbrains.com/plugin/1347-scala)

![Scala plugin](https://user-images.githubusercontent.com/401815/103236551-15a68780-493d-11eb-8a86-4885b98e94f2.png)

## Clone the repositories

    git clone --recursive https://github.com/ornicar/lila.git
    git clone https://github.com/ornicar/lila-ws.git

## Import the project

![Import project](https://user-images.githubusercontent.com/401815/103237147-ad58a580-493e-11eb-94f8-7d27ce866fde.png)

Thanks to the [build.sbt](https://github.com/ornicar/lila/blob/master/build.sbt) file in the root folder, IntelliJ is smart enough to automatically detect the project as an SBT Scala project. The import should set up your project correctly:

![sbt import](https://user-images.githubusercontent.com/401815/103237304-222bdf80-493f-11eb-80c7-aa9feb7111f9.png)

Dependencies should be installed automatically:

![Dependencies](https://user-images.githubusercontent.com/401815/103237836-c5c9bf80-4940-11eb-8237-c403a8c621f5.png)

Folder and project structure should be detected automatically:

![Folder structure](https://user-images.githubusercontent.com/401815/103237894-f90c4e80-4940-11eb-99e4-b8a1d1a24b15.png)

You will be asked to use the default code formatter. Please choose `scalafmt`:

![code formatter](https://user-images.githubusercontent.com/401815/103237986-3c66bd00-4941-11eb-9eba-27c4e8738e62.png)

Repeat for the `lila-ws` project.

## Using SBT

Working with the IDE and all its SBT features directly instead of the pure command line requires us to skip the [./lila](https://github.com/ornicar/lila/blob/master/lila) SBT wrapper functionalities.
We have to manually create our configurations once:

    cp .sbtopts.default .sbtopts
    cp conf/application.conf.default conf/application.conf

There are two integrated SBT tools you would want to use:
1. SBT Tool window
1. SBT Shell

![sbt tool](https://user-images.githubusercontent.com/401815/103238471-bcd9ed80-4942-11eb-9720-fcca259a8070.png)

![sbt shell](https://user-images.githubusercontent.com/401815/103238560-fd396b80-4942-11eb-85b9-b7c3b313cf72.png)


## Infrastructure

Instead of directly [installing redis and MongoDB as services](https://github.com/ornicar/lila/wiki/Lichess-Development-Onboarding#running-infrastructure) on your machine, you could spin up a dockerized version instead:

Write a `docker-compose.yml` file with those basic contents:

    version: "3.3"
    services:
      redis:
        image: "redis:alpine"
        ports:
          - 6379:6379
      mongo:
        image: mongo
        restart: always
        ports:
          - 27017:27017

Spin up the containers with:

    docker-compose up

And after use shut them down with: 

    docker-compose down

## Starting lila

1. `docker-compose up` to start the infrastructure 
1. `./ui/build` to build CSS and JS
1. Open up the `lila` and `lila-ws` projects in IntelliJ.
1. Go to the sbt shells in your IntelliJ projects and type `run`
1. Visit http://localhost:9663/ 

![Lichess.dev](https://user-images.githubusercontent.com/401815/103240328-f3663700-4947-11eb-8992-9a559be711ec.png)

## Debugging lila

In the IntelliJ settings find the sbt build tools options and check `Enable debugging`:

![sbt debugging](https://user-images.githubusercontent.com/401815/103239791-7e463200-4946-11eb-98f9-0fcecc06e7e0.png)

After reloading your project you should see a new button in your sbt shell:

![sbt debug button](https://user-images.githubusercontent.com/401815/103240011-09272c80-4947-11eb-9cf6-9b1d03b66a9b.png)

Click it and a debugger will be attached to your SBT shell. You can now set breakpoints and debug through your code!

![debugger](https://user-images.githubusercontent.com/401815/103240435-44762b00-4948-11eb-9b11-276775cd8e4e.png)