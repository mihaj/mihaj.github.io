---
layout: post
title: String hashing with MurMur algorithm
excerpt_separator: <!--more-->
author: Miha J.
tags: c# hashing
---

Hashing strings is fun. There is a [MurmurHash .NET](https://github.com/darrenkopp/murmurhash-net) which contains a .NET implementation for [MurmurHash algorithm](https://en.wikipedia.org/wiki/MurmurHash).

MurmurHash is a fast non-cryptographic algorithms which can be used to generate cache keys, for example `Memcached` or `Redis`. There is a good post on [Stack Exchange](https://softwareengineering.stackexchange.com/questions/49550/which-hashing-algorithm-is-best-for-uniqueness-and-speed) which goes into details by comparing many hashing algorithms.

By using a [MurmurHash .NET](https://github.com/darrenkopp/murmurhash-net) library you can create a function like this to generat hashes from strings:

```csharp
public static class HashFunctions
{
    /// <summary>
    /// Generate a non-cryptographic string hash by MurMur algorithm
    /// </summary>
    /// <param name="stringToHash"></param>
    /// <returns></returns>
    public static string GenerateStringHash(string stringToHash)
    {
        var hash = MurmurHash.Create128();
        var hashArray = hash.ComputeHash(Encoding.UTF8.GetBytes(stringToHash));

        return ConvertBytesToString(hashArray);
    }

    private static string ConvertBytesToString(in byte[] data)
    {
        Array.Reverse(data);
        var sBuilder = new StringBuilder();

        // Loop through each byte of the hashed data 
        // and format each one as a hexadecimal string.
        for (var i = 0; i < data.Length; i++)
        {
            sBuilder.Append(data[i].ToString("x2"));
        }

        // Return the hexadecimal string.
        return $"0x{sBuilder.ToString().ToUpper()}";
    }
}
```