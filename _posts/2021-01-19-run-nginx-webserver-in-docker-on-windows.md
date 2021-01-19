---
layout: post
title: Run Nginx Web in Docker on Windows
excerpt_separator: <!--more-->
author: Miha J.
tags: docker nginx
---

Sometimes you want to run the Nginx web server on Windows. Sometimes you want to run Nginx in Docker on Windows. Let me show you how you can do that.

To build a docker image, we need to create a `Dockerfile`. We need to select a Windows base image, such as Windows 2019 [Core](https://hub.docker.com/_/microsoft-windows-servercore) or [Nano](https://hub.docker.com/_/microsoft-windows-nanoserver) server images. Because official Nginx Windows binaries are not explicitly compiled for 64-bit operating systems, we can not use Windows Nano since it only supports 64-bit applications.

Let's start with the Windows 2019 Core server docker image with PowerShell already [installed](https://hub.docker.com/_/microsoft-powershell). We will need PowerShell to install the Nginx and set up the necessary configurations.

Create a folder `DockerTest` on your disk and create three files:

### Index.html
```
Hello from Nginx!
```

### nginx.conf
```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       80;

        location / {
          root c:/nginx/enabled-sites/html;
          index index.html index.htm;
          try_files $uri $uri/ /index.html =404;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

### Dockerfile
```dockerfile
FROM mcr.microsoft.com/powershell:lts-windowsservercore-1809 as build-stage

ENV NGINX_VERSION 1.19.6
ENV EnabledSitesPath=c:\\nginx\\enabled-sites\\html

SHELL ["pwsh", "-command"]
RUN Invoke-WebRequest "http://nginx.org/download/nginx-$($env:NGINX_VERSION).zip" -OutFile C:\nginx.zip
RUN Expand-Archive C:\nginx.zip C:\nginx
RUN Remove-Item "C:\nginx\nginx-$($env:NGINX_VERSION)\conf\*.conf" -Verbose
RUN New-Item -type directory "$($env:EnabledSitesPath)"
RUN Remove-Item C:\nginx.zip

WORKDIR c:\\nginx\\nginx-$NGINX_VERSION
COPY ./index.html c:/nginx/enabled-sites/html
COPY ./nginx.conf c:/nginx/nginx-$NGINX_VERSION/conf/nginx.conf
CMD ["nginx", "-g", "daemon off;"]
```
Now open up PowerShell and navigate to the DockerTest folder and run:

`docker build -f Dockerfile -t nginxhello:latest .`

After the image is built, run the docker container from that image:
`docker run --rm --name nginxhello -p 8888:80 -it nginxhello:latest`

Now you can navigate to your browser at `http://localhost:8888` and see your Index.html.

Because we use the Windows 2019 Core, the image size is pretty big, between 5-6GB. Alternatively, Windows Nano images are 10x smaller, and we will investigate installing Nginx on Windows Nano in a future post.