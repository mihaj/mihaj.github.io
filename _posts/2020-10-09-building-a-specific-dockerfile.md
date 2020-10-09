---
layout: post
title: Building specific dockerfile
excerpt_separator: <!--more-->
author: Miha J.
tags: docker nugget
---

To build a docker file, you would normally navigate to a folder and run:

```
docker build .
```

That will assume, that a `dockerfile` is located in a folder from where you run the command. A `dot` is a context in which the docker build will run. Sometimes a `dockerfile` is nested within subfolders and it's easeier for me to specify a docker file and keep the context. For example, let's assume we have a folder structure:

```
devops
  |->azure_pipelines.yaml
docs
  |->readme.md
src
  |->MyLibrary
      |->Class.cs
  |->UnitTests
      |->Class1Tests.cs 
  |->Service
      |->Dockerfile 
```

I want to run the docker build from the top folder to have a better control over the context. To instruct the docker build to run specific `dockerfile` you can write this instead:

```
docker build -f src/Service/Dockerfile .
```

I like this approach, since I can have the whole scope and context in actions within a Dockerfile.