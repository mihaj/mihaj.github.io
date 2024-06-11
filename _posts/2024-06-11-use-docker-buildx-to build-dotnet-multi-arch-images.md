---
layout: post
title: Use Docker buildx to build .NET multi-arch-images
excerpt_separator: <!--more-->
author: Miha J.
tags: docker, docker buildx, multi-arch images, multi architecture, arm64, amd64, dotnet, .NET
---

Nowadays, we run .NET 5,6,7,8, ... apps on the Linux operating system almost without any issues. We can use [Alpine MUSL linux](https://pkgs.alpinelinux.org/package/edge/main/x86/musl) [Docker image](https://hub.docker.com/_/alpine), put the self contained .NET executable in it, install few libraries and we are good to go.

Default architecture we normally use is `AMD64` with switch `--platform linux-must-64` in the `dotnet publish` command. We can run the containers in Kubernetes on Linux, or we run it on Linux VM. No problems there.

Lately we have some developers using MacBook M2 with ARM CPU architecture, and we were struggling to run this docker container on it. You will probably not run your production on MacBook, but you can already run your apps on [ARM virtual machines](https://azure.microsoft.com/en-us/updates/general-availability-armbased-vms-now-available-in-four-additional-azure-regions/), so it makes sense to do that.

And because of that, we decided we needed to built a multi-arch docker image of our .NET 8 application with [docker buildx](https://github.com/docker/buildx). We are using MUSL `Alpine Linux` docker image for [AMD64 and ARM64](https://hub.docker.com/_/alpine) and we build .NET assemblies outside of docker build process. You could use multi-staged Dockerfile build process.

These are the steps:

### 1. Enable Docker buildx

First you need to check if you have `buildx` module installed:

`docker buildx build --help`

Then create a new builder:

`docker buildx create --name my-app-builder --use --bootstrap`

This will run buildx container on your machine used to build the multi-arch docker images.

### 2. Build your app for ARM64 and AMD64

AMD64 CPU architecture:
```powershell
dotnet publish "./App.csproj" -c Release --framework net8.0 --self-contained True --runtime linux-musl-x64 -o "./publish/amd64" -p:PublishTrimmed=False -p:PublishSingleFile=True -p:IncludeNativeLibrariesForSelfExtract=True -p:DebugType=None -p:DebugSymbols=False -p:EnableCompressionInSingleFile=True
```
> To define the AMD64 CPU architecture use the `--runtime linux-musl-x64`.

ARM64 CPU architecture:
```powershell
dotnet publish "./App.csproj" -c Release --framework net8.0 --self-contained True --runtime linux-musl-arm64 -o "./publish/arm64" -p:PublishTrimmed=False -p:PublishSingleFile=True -p:IncludeNativeLibrariesForSelfExtract=True -p:DebugType=None -p:DebugSymbols=False -p:EnableCompressionInSingleFile=True
```
> To define the ARM64 CPU architecture use the `--runtime linux-musl-arm64`.

### 3. Change your docker file to dynamically use the TARGETPLATFORM parameter

```dockerfile
FROM --platform=$TARGETPLATFORM alpine:3.19 AS base
EXPOSE 80
ARG TARGETARCH

WORKDIR /app

COPY --chmod=755 ["publish/${TARGETARCH}/App", "./"]

ENTRYPOINT ["./App"]
```

> TARGETPLATFORM is injected from the `--platform linux/amd64,linux/arm64` below in the buildx command.

> TARGETARCH is defined by the build process and has values of `amd64` or `arm64`. Assemblies from step 2 are copied to the correct image.

### 4. Run the build process

Now you can run the build process below. It will build a multi-arch Docker image for `amd64` and `arm64`.
- `--tag` assignes a tag to an image.
- `--file` references a Dockerfile above.
- `--platform` defines the platforms you want to build your apps for.
- `--push` command automatically pushes the built docker image to your selected docker registry.

```powershell
docker buildx build --tag my.dockerhub.com/app:latest --file ./Dockerfile --platform linux/amd64,linux/arm64 --push .
```

### 5. Run docker container on ARM64

Now that you have an multi-arch docker image you can run it on ARM64 CPU system with:

```powershell
docker run -p 80:80 --name app --platform linux/arm64 -t my.dockerhub.com/app:latest
```

I hope this will help you to run your .NET apps in docker for multiple CPU architectures :)!