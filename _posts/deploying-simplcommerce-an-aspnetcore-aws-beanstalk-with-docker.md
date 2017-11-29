---
layout: post
title: Deploying SimplCommcere, an ASP.NET Core Application to AWS Elastic Beanstalk with Docker
---

### Introduction

As a .NET developer, my primary programming language is C# and Azure is the my first choice when thinking of the cloud. Recently, I have joined an interesting project that the customer requested us to use ASP.NET Core with Aurora database and host in AWS. They prefer AWS because they already have several applications there. They also want to optimize the hosting cost and prefer using open sources. Actually they requested us to develop application in PHP, but we have convinced them to use .NET Core. We said to them Microsoft have changed, .NET Core is awesome. It's fully open source and can run very well on Linux, bla. bla... Finally they agreed. :)

EC2 was not a preferred option because with EC2 we have to manage the VM: manually setup load balancing, scaling etc. AWS Elastic Beanstalk seems to match our need. It is quite similar to the Azure Web App. However, .NET Core 2.0 along with 1.1 are only supported on Beanstalk's Windows platform. A little disappointed.

ASP.NET Core run very well on Linux, how can we leverage that. Fortunately, Beanstalk supports deploying from docker containers. Sweet! with Docker containers, we can define our own run time environment. 

AWS Beanstalk with docker is nice. Thinking about SimplCommerce. I think it is great to see SimplCommerce run in AWS and fully on open source stack: ASP.NET Core, PostgreSQL, Linux, Docker. Let get started.

### First, the docker image

Although we have an automated docker build and the docker image is publish on docker hub, this docker image only for testing purpose only. For more convenient, entity framework migration, seeding data are done automatically and it is not suitable production.

So I have created a new docker image. You can find it in docker hub at [https://hub.docker.com/r/simplcommerce/simplcommerce-eb/](https://hub.docker.com/r/simplcommerce/simplcommerce-eb/)

### Step 1: Create PostgreSQL database

For SimplCommcere we choose PostgreSQL over Aurora or MySQL because I see PostgreSQL provider for Entity Framework Core is more stable than MySQL.

Login to the AWS Console(I am using Free Usage Tier), go to RDS and create a new PostgreSQL database. I choose Dev/Test for the use cases.

In the Specify DB details screen -> Instance specification. Check the option "Only enable option eligible for RDS Fre Usage Tier" and leave other as it is. In the settings section, filling your DB instance identifier (I filled simpldb), master username and password.

In the step 4: Configure advanced settings. Make sure the Public accessibility is Yes. Fill your database name (I filled simpldb) and leave other as default.

Then click on the button "Launch DB instance". Wait for a couple minutes for the instance to be ready.

Open a PostgreSQL compliance client tool such as "pgAdmin 4". Connect to the PostgreSQL instance you have just created. Execute [Schema_PostgreSQL.sql](https://github.com/simplcommerce/SimplCommerce/blob/master/src/Database/Schema_PostgreSQL.sql) script to create tables and [StaticData_PostgreSQL.sql](https://github.com/simplcommerce/SimplCommerce/blob/master/src/Database/StaticData_PostgreSQL.sql) to insert seeding data


### Step 2: Create S3 bucket

In SimplCommcere, admin can upload images, documents for products and categories. The rich text editor(summernote) also supports upload files. By default, SimplCommcere stores these uploaded files in the "user-content" folder under the wwwrooot.

Difference Azure Web App, in order to work in Beanstalk, the uploaded file have to be stored in an external storage like S3. Fortunately, SimplCommcere already support this and the "simplcommerce-eb" docker image has been configured to use Amazon S3.

In AWS Console, go to Amazon S3. Create a bucket and make sure that you grant the public read access to this bucket.

### Step 3: Create AWS Elastic Beanstalk application

What we need is the "Dockerrun.aws.json". Please see it below. 
```json
{
 "AWSEBDockerrunVersion": "1",
 "Image": {
    "Name": "simplcommerce/simplcommerce-eb",
    "Update": "true"
  },
 "Ports": [
   {
     "ContainerPort": "80"
   }
 ],
 "Logging": "/var/log"
}
```
It is quite simple. Where to pull the docker image? In our case is simplcommerce/simplcommerce-eb. It's a publish repository in docker hub, so we don't need to provide a credential for authentication. The container expose port 80. And the log should be written to /var/log folder of the host.

By default our container will behind an nginx proxy, and it's configured to accept request having the size of the body under 2M. I have configured to accept up to 6M.

In AWS Consone, go to Elastic Beanstalk, create new application. Enter application name and click create. Then create an environment (You can think environment here can be dev, test, staging, production). Choose web server environment, enter environment name. For the Platform select Docker and upload application code. You can package your own "Dockerrun.aws.json" or use the one I have created (here)[https://github.com/simplcommerce/SimplCommerce/blob/master/aws-beanstalk/simplcommerce-eb.zip]. Click on configure more options

In the software block in modify. This is where we will enter the environments which will be read by the container. We need to add the following variable:

| Property Name | Property Value |
|---------------|----------------|
|ConnectionStrings:DefaultConnection | The connection string to your PostgreSQL database |
|AWS:S3:AccessKeyId| Access key to S3 |
|AWS:S3:SecretAccessKey| Access key to S3 |
|AWS:S3:BucketName| The S3 bucket name |
|AWS:S3:RegionEndpointName| The region endpoint name of S3 |

The connection string should look like: 

`User ID={};Password={};Host={};Port=5432;Database={};Pooling=true;`






