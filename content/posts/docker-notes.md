+++
title = "Docker Notes"
date = 2018-11-28T14:11:55+01:00
draft = true
tags = []
categories = []
+++

## Install

On Ubuntu <https://docs.docker.com/install/linux/docker-ce/ubuntu/>

## Useful commands

Remove an image

```bash
docker stop <container-id>
docker rm <container-id>
docker image rm <image>
```

Pruning of unused images, containers, volumes, etc. See <https://docs.docker.com/config/pruning>

```bash
docker image prune <image>
docker volume prune
docker system prune
```
