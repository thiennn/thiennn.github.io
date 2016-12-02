---
layout: post
title: Dockerizing a real world asp.net core application
---

<div class="alert alert-warning" role="alert">
 This is the second version of dockerizing simpcommerce. Read the first version [here](/dockerizing-real-world-aspnetcore-application-original/) to have a full story
</div>

The day after [successful dockerizing simplcommerce](/dockerizing-real-world-aspnetcore-application-original/), I started to re-look at the approach, the code and I also received a bug that the container fail to start again after stopping. 
The proudest thing of my approach is only one command needed to run the entire the application including the database. But it also reveals several drawbacks:

-	The dockerfile is big, and it take long time to build around 15 minutes in dockerhub
-	Postgres has its own way to initialize the container. I have extended that process to do entity framework migration, import static data, and call dotnet run to launch the website. But process is only run one, on the first start of the container, so this is the root cause of the bug I have mentioned above
-	Putting the database and the website into one box generally is not a good practice.

I have decided to make change. The first thing I do is separate the database and the website. For the database I use the default images of postgres https://hub.docker.com/_/postgres/ without any customization

“Separating what changes from what stays the same” always a good practice. So it’s a good idea separate the source code from the sdk. With this in mind I created the Dockerfile call simpl-sdk. I need dotnet:1.1.0-sdk-projectjson, nodejs, gulp-cli and postgresql client. So it make scenes to start from microsoft/dotnet:1.1.0-sdk-projectjson then install these stuff.

## simpl-sdk Dockerfile

{% gist 04746416ba08c697e4713b08ea3b7501 %}
Create a github repository and go to docker hub create automated build repository similar to [the way I did for simpcommerce](/dockerizing-real-world-aspnetcore-application-original/)

## SimplCommerce Dockerfile
Then the Dockerfile for simpcommerce then become very small and clean. 
{% gist f1fee8650e42e4ccba5f9a5f5b2ef20b %}
From the simpl-sdk, copy the source code in, restore nugget packages, build entire the projects, call `gulp copy-modules` to copy the build output of modules to the host. Copy and set the entry point. 


## The entry point
{% gist bc6a688d733b3c3d743185ed45d3fde3 %}
At this time the connection to database is ready run `dotnet ef database update` to run migration, and use psql connect to the database if no data found in database then import static data, then call `dotnet run` to start the app

Look simple and straight forward huh. But it took me a lot of time to make it run smoothly. First I don’t familiar with writing a shell script, I am a windows guy. Second, also not familiar with psql. Third some weird difference between linux and windows.
The first error I got when rum the images was 
“panic: standard_init_linux.go:175: exec user process caused "no such file or directory" [recovered]
        panic: standard_init_linux.go:175: exec user process caused "no such file or directory” 
What the hell is that? After some googling, I fixed by changing the line ending in the docker-entrypoint.sh from CRLF to LF. In Notepad++ select Edit -> EOL Conversion

The second error is 
“docker: Error response from daemon: invalid header field value "oci runtime error: container_linux.go:247: starting container process caused \"exec: \\\"/docker-entrypoint.sh\\\": permission denied\"\n".”
Something related to permission huh. Continue googling, then able to fix by adding
`RUN chmod 755 /docker-entrypoint.sh` before the `ENTRYPOINT ["/docker-entrypoint.sh"]`

## Bonus - Some usefull psql commands
