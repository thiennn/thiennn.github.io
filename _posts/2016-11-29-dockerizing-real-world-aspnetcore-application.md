---
layout: post
title: Dockerizing a real world asp.net core application
---

As you might know. I am developing an open source, cross platform and modularized ecommerce system built on .NET Core which called [simplcommerce](https://github.com/simplcommerce/SimplCommerce). 

![SimplCommerce](/images/simplscreenshot.png "SimplCommerce")

The system now has basic features of an ecommerce system like: manage catalog, shopping cart, orders, review products etc. Generally, it is enought for simple stores. Recently, many people come to ask me questions: doesn't run on Linux? can I deploy them to a container?
I think it's a good idea to have a docker image for simplcommerce, I started to think about it. And below are 2 things that I want to archive:

- It should be as simple as possible to for anybody who want to try simplcommerce. We have a demo site [here](http://demo.simplcommerce.com), but it will be more interesting to have it run on local, especially in a docker container.
- The image should be created automatically every time we make a change on the github repo.

Fascinating! But, to be honest. I am new to docker and Linux, I have some general knowldege but no real experience. After reseaching for a while, reading many articles, tutorials about put an asp.net core application to a docker container. But most of them are very basic just a hello world application. So it took me a while to figure how to dockerizing simplcommerce. Last night, finally I dit it. Now you just simply run one single command **`docker run -d -p 5000:5000 simplcommerce/nightly-build`**, waiting for some minutes. Then open your brower, type localhost:5000 and everything just work Yahoo!!!!. In this blog post I will describe how I did it.

Basically, challenges I need to resolve are:

 1. How to automate this process?
 2. How to deal with database? How to run entity framework migration? How to import some static data? (simplcommerce needs some data in the database before it start)
 3. How to run a gulp task there. The simplcommerce are composed by modules and a gulp task need to be run to copy modules to the host.

For the first challenge. In docker hub, I found that we can create "created automated build" from a github repository. Yeah, that's cool. So, just create one and then connect to simplcommerce github repository. 

![Create automated build](/images/docker-automated-build.png "Create automated build")

In the build setting, I check the option to make the build happen on pushes, and specify the location of the dockerfile.

![Automated build setting](/images/docker-automated-build_setting.png "Automated build setting")

For the database, normally people will go with a separate container and may leverage docker compose to wire them together. But this is complicated to me :D. This is not for production so I want thing to be as simple as possible. I decided to buddle every to one box. From the ubuntu, I installed postgres, then donetcore sdk, then nodejs. But I faced problems with staring postgres and its complidated permission mechanism. Yep! it is really complicated to a newbie as me. Getting stuck for a while. Then use the postgres as the base image, doing more research, eventually I can make it worked. You can see the dockerfile [here](https://github.com/simplcommerce/SimplCommerce/blob/docker/Dockerfile). With postgres image, we can inject custom commands to run after the database started by putting *.sh or *.sql file in a directory called "docker-entrypoint-initdb.d". Basically the dockerfile will do the following steps:

 1. From postgres:9.5
 2. Install some stuff that donetcore sdk and nodejs depend on
 3. Install dotnetcore sdk
 4. Install nodejs
 5. Add custom commands to "docker-entrypoint-initdb.d" directory in the image. Here I use a shell script "dockerinitcontainer.sh"
 6. Copy the simplcommerce source code to the image
 7. Build the all the .netcore project
 8. Install gulp and run "copy-modules" task to copy modules to the host

After the containers started, and the postgres engine started. The file "[dockerinitcontainer.sh](https://github.com/simplcommerce/SimplCommerce/blob/docker/dockerinitcontainer.sh)" is called which will

 1. Run entity framework migration
 2. Import some static data by using psql
 3. Finally start the app.

Yeah, a real world dotnet app run perfectly on a docker container. One command to setup everything and here we go.

This is my first step to docker and linux, It may be not perfect. So if you can give me suggestion, I will be more than happy.
