---
layout: post
title: Install application to IIS web server through Powershell
excerpt_separator: <!--more-->
author: Miha J.
tags: powershell, IIS
---
<!--more-->
You can deploy a web application to IIS with Powershell. Actually, you can do a lot with the tools `APPCMD`, `msbuild` and `msdeploy`. The script is handy for your automated deployments and CI/CD pipelines. The complete script is below.

There are two optional sections below for binding SSL certificates and to add an IP address to the network interface if you need them.

```powershell
# Setting variables
$interfaceIndex = 8
$ip = "192.168.111.111"
# Creating Application Pool in IIS.
%systemroot%\system32\inetsrv\APPCMD add apppool /name:AppPool-Demo
# Creating a site in IIS.
%systemroot%\system32\inetsrv\APPCMD add site /name:Demo /physicalPath:"c:\inetpub\wwwroot\Demo" /bindings:"https/$ip:443:"
# Assign an App pool to the site. 
%systemroot%\system32\inetsrv\APPCMD set site /site.name:Demo /[path='/'].applicationPool:AppPool-Demo
# Then we build our .NET project. and create the package for IIS deployment
msbuild "Demo.csproj" /T:Package /P:Configuration=Release /P:PackageLocation="c:\Package\Demo.zip"
# Deploy a package to IIS.
msdeploy -verb:sync -source:package="c:\Package\Demo.zip" -dest:auto -setParam:name='IIS Web Application Name',value='Demo' -skip:Directory=App_Data

# OPTIONAL: assign a SSL certificate to the app
# Import SSL certificate to the Windows Certification store
certutil -f -p "MySSLCertName" -importpfx "MySSlCert.pfx"
# Get the certificate thumbprint from the Windows Cert Store (below is an example)
$sslThumbprint = "ca4687b2c4499cf057d766cd56cd46c4ddf2a3ea"
# Add SSL certificate to the IP:PORT with thumbprint and generated APPID as GUID
netsh http add sslcert ipport=$(ip):443 certhash=$sslThumbprint appid="{$([guid]::NewGuid())}"

# OPTIONAL: You can run multiple IIS applications on one server with the same port by creating an IP to the network interface.
# You can bind that IP to the IIS application or bind the SSL certificate to it.
powershell New-NetIPAddress -InterfaceIndex $interfaceIndex -IPAddress $ip -PrefixLength 24
```

I hope you find this helpful.