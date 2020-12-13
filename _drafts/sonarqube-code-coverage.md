---
layout: post
title: Install an IIS application through Powershell
excerpt_separator: <!--more-->
author: Miha J.
tags: powershell
---

When deploying and application to IIS you have an option to script it. That is also useful for CI/CD pipelines.

```powershell
# Creating Application Pool in IIS.
%systemroot%\system32\inetsrv\APPCMD add apppool /name:AppPool-Demo
# Creating a site in IIS.
%systemroot%\system32\inetsrv\APPCMD add site /name:Demo /physicalPath:"$($IISInstallationPath)Demo" /bindings:"https/127.0.0.1:443:"
# Assign an App pool to the site. 
%systemroot%\system32\inetsrv\APPCMD set site /site.name:Demo /[path='/'].applicationPool:AppPool-Demo
# Then we build our .NET project.
msbuild "Demo.csproj" /T:Package /P:Configuration=Release /P:PackageLocation="c:\Package\Demo.zip"
# Deploy a package to IIS.
msdeploy -verb:sync -source:package="c:\Package\Demo.zip" -dest:auto -setParam:name='IIS Web Application Name',value='Demo' -skip:Directory=App_Data
```