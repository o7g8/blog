+++
title = "Running Dyalog APL in Docker/Linux"
date = 2018-11-28T11:14:26+01:00
draft = true
tags = ["docker", "Linux", "APL"]
categories = []
+++

In the article I use Ubuntu 16.04.

I assume that Docker is already installed according to [Get Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

Get inspiration from:

* APL Serverless: <https://github.com/mvranic/kubeless-apl-demo>, <https://github.com/kubeless/kubeless/compare/master...mvranic:master>

* Dockerfiles:
  * [JSONServer Dockerfile](https://github.com/mvranic/JSONServer/blob/master/Docker/Dockerfile).
  * <https://github.com/mvranic/kubeless/blob/master/docker/runtime/apl/Dockerfile.17.0>

* <https://hub.docker.com/r/dyalog/dyalog/>

* <https://hub.docker.com/r/dyalog/miserver/>

* <https://hub.docker.com/r/dyalog/jsonserver/>

Use [phusion/baseimage](https://hub.docker.com/r/phusion/baseimage/) as a base image because of compactness and correct handling of zombie processes. Another candidate is [Minimal Ubuntu](https://blog.ubuntu.com/2018/07/09/minimal-ubuntu-released).

Dyalog 17.0 linux can be found at R:\dyalog\installprograms\170\Linux\
Look on Docker Hub for non-commercial Linux Docker images like <https://hub.docker.com/r/dyalog/dyalog/> for APL 17.1 .