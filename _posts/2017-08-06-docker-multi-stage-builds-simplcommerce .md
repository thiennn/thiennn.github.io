---
layout: post
title: How did I reduce the docker image of SimplCommerce from 900M to 145M using docker multi-stage builds
---

### Introduction

Always looking for better ways to do thing is one of our principles for building <a href="https://github.com/simplcommerce/SimplCommerce" target="_blank">SimplCommerce</a>. Last week, I made another refactoring for our automated docker build by applying multi-stage docker builds. We were able to reduce the SimplCommerce image size from 900M to 145M (compressed). In this blog post, I will describe how we did that.

Let review the situation: We want to run a docker automated build for every change happens on master branch. 

- [This is our first version](http://thienn.com/dockerizing-real-world-aspnetcore-application-original/)
- [This is second version](http://thienn.com/dockerizing-real-world-aspnetcore-application/)

One of our problems is that the docker image size is quite big around 900M (compressed). Because it have to include everything so that it can build the application by itself such as dotnet core sdk, nodejs, gulp, and all the source code. 

Recently, Docker has introduced <a href="https://docs.docker.com/engine/userguide/eng-image/multistage-build/" target="_blank">multi-stage build</a>. 

> With multi-stage builds, you use multiple FROM statements in your Dockerfile. Each FROM instruction can use a different base, and each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don’t want in the final image

By applying multi-stages build I have divided the SimplCommerce Dockerfile into 2 stages:

{% gist a25ca0c2459f3aa5534fd8de73e44351 %}

### The first stage

In the first stage, I use the <a href="https://hub.docker.com/r/simplcommerce/simpl-sdk" target="_blank">simplcommerce/simpl-sdk</a> as a base image. Basically, it is an image contains dotnet core sdk, nodejs and gulp-cli

- Because the SimplCommerce source code is using MSSQL, I run some SED commands (find and replace) to make it work for PostgeSQL.

- Install gulp and run "copy-modules" gulp task to copy all the modules to the host. SimplCommere is built based on <a href="https://www.codeproject.com/Articles/1109475/Modular-Web-Application-with-ASP-NET-Core" target="_blank">modular architecture</a>

- I also need to delete all added migrations and run add migration again the because the difference between MSSQL and PostgreSQL. 

- `dotnet ef migrations script -o dbscript.sql` will generage sql script for the database schema. 

- Publish the website.

- However, The generated sql file is <a href="https://www.postgresql.org/message-id/201003310441.o2V4fEMm048826@wwwmaster.postgresql.org" target="_blank">utf-8 encoded which having issue when run by psql command</a>.

> The psql command gives syntax errors when fed .sql files that are saved with
> encoding "UTF-8" and contain a BOM (byte order marker)

- Remove the BOM in sql script by SED commands

### The second stage

The runtime only microsoft/aspnetcore:2.0.0-preview2-jessie image is used as the base. We also need to install postgresql-client in order to execute sql script.

I have named the fist stage "build-env" by adding an `AS build-env` to the FROM instruction. We can use that name in the COPY instruction

`RUN chmod 755 /docker-entrypoint.sh` to set execute permission for the docker-entrypoint.sh

When the docker run it will execute the docker-entrypoint.sh which will connect to PostgreSQL create database, tables and insert some pre-defined data if needed.

{% gist 02b7dbdb199c65dfef45ea9fac2bc6c1 %}

### Note

At this time <a href="https://github.com/docker/hub-feedback/issues/1039" target="_blank">Docker Hub has supported multi-stage builds</a>. Fortunately, Docker Cloud has supported it.

![SimplCommcere  Docker Cloud](/simplcommerce-docker-cloud.png "SimplCommcere  Docker Cloud")
