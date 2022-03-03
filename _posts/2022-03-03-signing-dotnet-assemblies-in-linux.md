---
layout: post
title: Signing .NET assemblies in Ubuntu Linux
excerpt_separator: <!--more-->
author: Miha J.
tags: .NET, net6, c#, code signing, code signing certificate
---

Signing assemblies in Ubuntu Linux with a code signing certificate is relatively straightforward. I suggest that we start by building out the application.

For Linux, we can use something like this:

`dotnet publish MyProject.csproj --configuration Release --framework net6.0 --output Publish --runtime linux-x64`

That will build our application with all DLL dependencies and the library it needs to run. Next, let's go to the Ubuntu Linux terminal and install [Mono development tools](https://packages.debian.org/sid/mono-devel) and **OpenSSL package**.

```bash
sudo apt-get install libssl-dev mono-devel
```

That will install the' signcode' tool to sign the assembly and ˙openssl˙ tool to convert our PFX certificate to the SPC and PVK files. Let's do that next:

```bash
# extract the certificate key from the PFX
openssl pkcs12 -in mycert.pfx -nocerts -nodes -out key.pem
# convert key from PEM to PVK
openssl rsa -in key.pem -outform PVK -pvk-strong -out signing.pvk
# extract certificate from the PFX
openssl pkcs12 -in mycert.pfx -nokeys -nodes -out cert.pem
# convert cert from PEM to SPC (DER output)
openssl crl2pkcs7 -nocrl -certfile cert.pem -outform DER -out signing.spc
```

Now we can sign our DLL assembly with:

```bash
signcode -spc signing.spc -v signing.pvk -a sha256 -$ commercial -n MyProj -t http://timestamp.sectigo.com/ -tr 10 /mnt/c/MyProject.dll
```

Because we can not sign a single self-contained .NET executable
```bash
System.NotSupportedException: Cannot sign non PE files, e.g. .CAB or .MSI files (error 3).
  at Mono.Security.Authenticode.AuthenticodeBase.ReadFirstBlock () [0x00027] in <e6d76f81f5874e0ba9bb032cf8be34bf>:0
  at Mono.Security.Authenticode.AuthenticodeBase.GetHash (System.Security.Cryptography.HashAlgorithm hash) [0x0000c] in <e6d76f81f5874e0ba9bb032cf8be34bf>:0
  at Mono.Security.Authenticode.AuthenticodeFormatter.Sign (System.String fileName) [0x00013] in <e6d76f81f5874e0ba9bb032cf8be34bf>:0
```
we can write a script to sign all DLLs in a folder like this:

```bash
for i in $(find /mnt/c -maxdepth 1 -type f -path '*.dll')
do
    signcode -spc signing.spc -v signing.pvk -a sha1 -$ commercial -n MyProj -t http://timestamp.sectigo.com/ -tr 10 ${i}
done
```
Hint: You can put the above script into a `sign.sh` file and run it.

Now if you want to validate the signing you can run this command:

```bash
chktrust MyProject.dll
Mono CheckTrust - version 6.8.0.105
Verify if an PE executable has a valid Authenticode(tm) signature
Copyright 2002, 2003 Motus Technologies. Copyright 2004-2008 Novell. BSD licensed.

SUCCESS: MyProject.dll signature is valid
and can be traced back to a trusted root!
```

If you are using self-signed certificates (strongly not recommended), you will need to add the certificate to Ubuntu trusted certificate store:

```bash
sudo mkdir /usr/share/ca-certificates/extra
# convert pfx to crt
openssl pkcs12 -in mycert.pfx -out signing.crt -nokeys -clcerts
# copy crt to cert store folder
sudo cp signing.crt /usr/share/ca-certificates/extra/signing.crt
# update certificate store with new certs
sudo dpkg-reconfigure ca-certificates
sudo update-ca-certificates
```