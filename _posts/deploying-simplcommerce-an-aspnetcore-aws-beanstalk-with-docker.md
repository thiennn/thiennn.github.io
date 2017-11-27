---
layout: post
title: Deploying SimplCommcere, an ASP.NET Core Application to AWS Elastic Beanstalk with Docker
---

### Introduction

As a .NET developer, my primary programming language is C# and Azure is the my first choice when thinking of the cloud.Recently, I have joined an interesting project that the customer requested us to use ASP.NET Core with Aurora database and host in AWS.

EC2 was not a preferred option because with EC2 we have to manage the VM, manually setup load balancing, scaling etc. After some research I found that AWS Beanstalk can match our need. It is quite similar to the Azure Web App. Reading more documents, I discovered that Amazon only support ASP.NET Core on Windows, that mean our customer have to pay for the Windows server license which is not very cheap :).

ASP.NET Core run very well on Linux, how can we leverage that. Fortunately, Beanstalk supports deploying from docker containers. Great! With Docker containers, we can define our own run time environment. The project was completed successfully. 

AWS Beanstalk with docker is nice, Can we deploy SimplCommcere into it? Why not? In this post. I will describe how I deploy SimplCommcere to AWS Beanstalk with Docker.

For your simplicity, I have made a small adjustment and built docker image. You can jump to step 3 if you want to go fast.  

### Step 1: Switch from MS SQL to PostgreSQL

By default, SimplCommerce use MS SQL Server. In the context of this post, we want to reduce the hosting cost as much as posible, so we will switch to PostgreSQL. For SimplCommcere we choose PostgreSQL over Aurora or MySQL because I see PostgreSQL provider for Entity Framework Core is more stable than MySQL.

In Visual Studio, open the WebHost project file 'SimplCommerce.WebHost.csproj'. Replace 

`<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="2.0.0" />`

with

`<PackageReference Include="Npgsql.EntityFrameworkCore.PostgreSQL" Version="2.0.0" />`

Save the file and wait for a couple of seconds for the package restore. Then to the Program.cs, line 32, replace the text `UseSqlServer` with `UseNpgsql`. Similarly at the line 153 in ServiceCollectionExtensions.cs. 

Delete all the file in SimplCommerce.WebHost/Migrations/. Then build the solution and add Migration again. (The existing migration is for MS SQL Server).

Run Script-Migration command to generate sql script for the database schema.

### Step 2: Make SimplCommcere stateless

In SimplCommcere, admin can upload images, documents for products and categories. The rich text editor(summernote) also supports upload files. By default, SimplCommcere store these uploaded file in the "user-content" folder under the wwwrooot.
Difference Azure Web App, in order to work in Beanstalk, the uploaded file have to be stored in an external storage like S3. Fortunately, SimplCommcere already support that.

All you need to do is switching from StorageLocal (default) to StorageAmazonS3. In Visual Studio or in Windows Explorer, cut the module.json in the "SimplCommerce.Module.StorageLocal" project and paste it into the SimplCommerce.Module.StorageAmazonS3. Then modify the name of the module in that file

```json
{
  "name": "storageAmazonS3",
  "fullName": "SimplCommerce.Module.StorageAmazonS3",
  "version": "1.0.0"
}
```

Without the module.json file, the module will not be copied to the WebHost.

### Step 3: Build the docker image

### Step 4: Prepare the database

Login to the AWS Console(I am using Free Usage Tier), go to RDS and create a new PostgreSQL database then choose Dev/Test for the use cases.

In the Specify DB details screen -> Instance specification. Check the option "Only enable option eligible for RDS Fre Usage Tier" and leave other as it is. In the settings section, filling your DB instance identifier (I filled simpldb), master username and password.

In the step 4: Configure advanced settings. Make sure the Public accessibility is Yes. Fill your database name (I filled simpldb) and leave other as default.

Then click on the button "Launch DB instance". Wait for a couple minutes for the instance to be ready.

Open a PostgreSQL compliance client tool (I use pgAdmin 4). Connect to the PostgreSQL instance you have just created. You can find the endpoint of your PostgreSQL instance in AWS console. Execute 

Run sql script to create tables and insert predefined data

### Step 5: Create S3 bucket

### Step 6: Create beanstalk application

### Deploy updates



