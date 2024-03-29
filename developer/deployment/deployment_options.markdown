---
title: "Deployment Options"
section: deployment
---

## Overview

This guide will discuss the requirements and options for deploying Colibri
in production and staging environments. After reading it, you should be
familiar with:

-   Common server configurations used when deploying Colibri.
-   How memory availability affects Colibri's performance and scalability
-   Typical memory footprints for Colibri servers.

Colibri can be deployed like any other Rails application using any
combination of the usual software components like Apache, Nginx,
Unicorn, Passenger, Ruby, Capistrano etc.

Normally the first concern when deploying your store is choosing the
correct software stack, server sizes and roles.

Using the [Colibri Deployment Service](deployment-service) you can
avoid the hassle of choosing and configuring all the individual
components, and jump straight to getting your application deployed using
our preconfigured and tested software stack.

***
This guide assumes basic familiarity with Colibri. Please refer to
the [Getting Started Guide](getting_started_tutorial) for details on how to
get up and running for Colibri. You will have an easier time understanding
the extra complications of a production setup.
***

### Server Sizing & Configurations

While most smaller Colibri installations can operate extremely well on
relatively modest single server configurations, stores that need to
handle a larger number of concurrent users frequently chose a
multi-server configuration to distribute the workload and simplify
server configurations.

***
Determining how many concurrent users any server configuration is
capable of handling is entirely dependant on the specific application
and its use of extensions and customizations. All server sizing
guidelines provided here are purely estimates based on basic Colibri
instances.
***

### Memory (RAM) requirements

Colibri's scalability (like most Rails applications) is directly affected
by number of application worker processes available to handle requests
as they arrive. The number of worker processes you can configure for a
given server is directly related the amount of RAM available.

***
The **Colibri Deployment Service** allows you to configure the
number of unicorn workers deployed on each application server.
***

### Single Server

Single server configurations are well suited to low volume Colibri
production installations, or staging / qa environments. Single server
configurations include both the application and database software
components running on a single host, and generally need to be configured
to run less workers to allow some free memory for the database
component.

#### Suggested initial memory & worker values

These values are guidelines for the minimum requirements for a single
server production environment. Staging and QA can perform acceptably
with less resources, generally half of what is listed below.

|Colibri Version|RAM (minimum)|Unicorn Workers|
|------------:|------------:|--------------:|
|1.0.0|1024MB|2-4|
|0.70.x|1024MB|2-4|
|0.60.x|1024MB|2-4|
|0.11.x|512|2-3|

!!!
Adding too many workers for the amount of available memory will
have a negative effect on overall performance so it's important to get
the worker count correct for your application / memory combination.
!!!

### Multi-Server

When you separate your Colibri application on multiple servers, generally
one or more servers are configured as dedicated application servers and
one server is configured as a dedicated database server.

#### Suggested initial memory & worker values

These values are guidelines for the minimum requirements for each
Application Server in a production environment.

|Colibri Version|RAM (minimum)|Unicorn Workers|
|------------:|------------:|--------------:|
|1.0.0|512MB|3-5|
|0.70.x|512MB|3-5|
|0.60.x|512MB|3-5|
|0.11.x|512MB|2-4|

Database servers for use in multi-server configurations generally
require a minimum of 512MB RAM when serving one or two applicaiton
servers, however this should be increased when more or larger capacity
application servers are used.
