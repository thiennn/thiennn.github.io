---
layout: post
title: Dockerizing a real world asp.net core application
---

<p align="center">
   <a href="https://hub.docker.com/r/simplcommerce/nightly-build" target="_blank">
      <img src="/images/docker_simpl_s.png" alt="SimplCommerce on Docker" />
   </a>
</p>

<div class="alert alert-warning">
 This is the second version of dockerizing simpcommerce. Read <a href="/dockerizing-real-world-aspnetcore-application-original">the first version</a> to have the full story
</div>

The day after [successfully dockerizing simplcommerce](/dockerizing-real-world-aspnetcore-application-original/), I started to re-look at the approach, the code. I also received a bug report that the container fail to start again after stopping. 
The proudest thing of what I have done is that It only need one command to run the entire the application including the database in one container. But it revealed several drawbacks:

-	The dockerfile is big, and it take long time to build. Around 15 minutes in dockerhub
-	Postgres has its own way to initialize the container. I have extended that process to do entity framework migration, import static data and call `dotnet run` to launch the website. But this process is only run once, after the container started for the fist time, so this is the root cause of the bug I have mentioned above
-	Putting the database and the website into one box generally is not a good practice.

I have decided to make changes. The first thing I do is separating the database and the website. For the database I use the default images of (postgres)[https://hub.docker.com/_/postgres/] without any customization.

**_“Separating what changes from what stays the same”_** is always a good practice. So it’s a good idea to separate the source code from the sdk. With this in mind I created [simpl-sdk](https://hub.docker.com/r/simplcommerce/simpl-sdk/) docker image. I need dotnet core 1.1 project json sdk, nodejs, gulp-cli and postgresql client. It make scene to start from microsoft/dotnet:1.1.0-sdk-projectjson then install other stuffs.

## simpl-sdk Dockerfile
{% gist 04746416ba08c697e4713b08ea3b7501 %}
I created a github repository for it and went to docker hub to created an automated build repository similar to [the way I did for simpcommerce earlier](/dockerizing-real-world-aspnetcore-application-original/)

## SimplCommerce Dockerfile
Now the Dockerfile for simpcommerce become very small and clean. 
{% gist f1fee8650e42e4ccba5f9a5f5b2ef20b %}
From the simpl-sdk, copy the source code to the image, restore nugget packages, build entire the projects, call `gulp copy-modules` to copy the build output of modules to the host. Copy and set the entry point. 

## The entry point
{% gist bc6a688d733b3c3d743185ed45d3fde3 %}
At this time the connection to database is ready. Run `dotnet ef database update` to run migration, and use psql connect to the database, if no data found in database then import static data. Finally call `dotnet run` to start the app

Look simple and straight forward huh. But it took me a lot of time to made it run smoothly

- First I am not familiar with writing a shell script. 
- Second, I am also not familiar with psql. 
- Third some weird difference between linux and windows.

#### The first error that I got when starting the container was 
_“panic: standard_init_linux.go:175: exec user process caused "no such file or directory" [recovered]
        panic: standard_init_linux.go:175: exec user process caused "no such file or directory”_ 

What the hell is that? After some googling, I fixed by changing the line ending in the docker-entrypoint.sh from CRLF to LF. In Notepad++ select Edit -> EOL Conversion

#### The second error was 
_“docker: Error response from daemon: invalid header field value "oci runtime error: container_linux.go:247: starting container process caused \"exec: \\\"/docker-entrypoint.sh\\\": permission denied\"\n".”_

Something related to permission huh. Continue googling, then I was able to fix it by adding
`RUN chmod 755 /docker-entrypoint.sh` before the `ENTRYPOINT ["/docker-entrypoint.sh"]`

## Bonus - Some usefull psql commands
Connect to a server run a query and print the out to a file  
`echo 'select count(*) from "Core_User";' | psql -h simpldb --username postgres -d simplcommerce -t > /tmp/varfile.txt`  
-h: host, -d: database, -t: to get just the tuple value  

Run an sql script  
`psql -h simpldb --username postgres -d simplcommerce -a -f /app/src/Database/StaticData_Postgres.sql'`  
-h: host, -d: database, -a: echo all queries from scripts, -f: file  

Connect to a server  
`psql -h simpldb --username postgres`  

### When connected
`\l` : list all databases   
`\c dbname` : connect to dbname database  

### In a database
 `\i path.sql` : exe sql  
 `\dt` : list table  
 `select * from "Core_User";` execute a query, the semicon at the end is required, table name is case sensitive  

### Quick psql
`\q` : quick
