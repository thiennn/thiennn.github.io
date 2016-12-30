---
layout: post
title: Change primary key for ASP.NET Core Identity and more
---

By default ASP.NET Core Identity use a string value for the primary keys. In [SimplCommerce](https://github.com/simplcommerce/SimplCommerce) we decided to use a long value instead.
We also change the name of identity tables and run some custom code everytime a user login in to the system. This post I am going to show you how we did this.

