---
subtitle: '<span id="h.30j0zll" class="anchor"></span>Version 2016040102'
title: |
    <span id="h.gjdgxs" class="anchor"></span>AdditionSecurity Messaging Gateway\
    QuickStart: Docker Deploy
---

Overview
========

This document walks through the configuration and deployment of the AdditionSecurity Messaging Gateway using Docker.

Requirements
============

You will need the following:

-   Downloaded AdditionSecurity Messaging Gateway software from the AdditionSecurity customer portal

-   A suitable system running Docker

It is also assumed you have basic understanding of:

-   Docker configuration and usage basics

Deployment Topology
===================

The target topology for this quickstart:

-   A Docker container running the Message Gateway application, listening to incoming HTTP requests on TCP port 5000 (exposed via Docker)

Preparing the Message Gateway Deployment Files
----------------------------------------------

Docker deployments use the contents provided in the downloaded Message Gateway distribution file, along with the Dockerfile contained in the deploy\_docker/ subfolder/subdirectory.

For successful deployment you will need four files:

-   asgw.jar - the AdditionSecurity Message Gateway application

-   Dockerfile - Docker image build directives

-   config.properties - configuration used for the Message Gateway

-   Definitions.json - meta-data file used by the Message Gateway application

The Message Gateway distribution .zip file downloaded from the AdditionSecurity customer portal contains the necessary base files. **You must create an appropriate config.properties file before the Docker image can be built.**

### Step 1: Edit the config.properties file

You can edit an existing config.properties file (e.g. the sample file included with the AdditionSecurity Message Gateway software distribution), or create a new empty file.

The particular configuration depends upon your desires for message transform format and eventual output. Please read the Messaging Gateway Operations Guide to determine a proper configuration.

The relevant configuration items particular to a Docker deployment are:
```
port=5000

input=protobuf
```
Full details of these configuration values, and other available configuration values, are available in the AdditionSecurity Message Gateway Operations Guide available for download from the AdditionSecurity customer portal.

### Step 2: Locate all the files together

The updated config.properties file, along with other necessary files, should be placed into a single directory:

-   asgw.jar

-   Dockerfile

-   config.properties

-   definitions.json

Building an Image in Docker
===========================

In the directory containing the Dockerfile and other files, execute the following docker command to construct a Docker image named “asgw”:
```
docker build -t “asgw” .
```
A new image will be built. The output will resemble:
```
Sending build context to Docker daemon 3.404 MB

Step 1 : FROM java:8-jre-alpine

---&gt; 7cd507743d7d

Step 2 : MAINTAINER support@additionsecurity.com

---&gt; Running in 5ff167b6a863

---&gt; b72861a25e07

Removing intermediate container 5ff167b6a863

Step 3 : ADD asgw.jar config.properties definitions.json /

---&gt; 3834e931de50

Removing intermediate container 34da186274ac

Step 4 : ENTRYPOINT java -jar asgw.jar config.properties

---&gt; Running in 9f90fae9d83f

---&gt; 9c41a5e2904e

Removing intermediate container 9f90fae9d83f

Step 5 : EXPOSE 5000

---&gt; Running in 7792d66a8d3b

---&gt; 12f43353f873

Removing intermediate container 7792d66a8d3b

Successfully built 12f43353f873
```
Running the Docker Image
========================

After previously building the “asgw” image, you can now run it:
```
docker run -p 5000:5000 -t “asgw”
```
You should see the successful Message Gateway startup:

ASGateway 1.1

- Upload/temp dir: /tmp

- Definitions: (Definitions not loaded)

- Input: AddSec Protobuf

- Transform: KVP

- Output: Sinkhole

Ready to receive requests on port 5000

The Docker container is now running and exposing an HTTP listener on TCP port 5000.
