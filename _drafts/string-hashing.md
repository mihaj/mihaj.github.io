---
layout: post
title: String hashing with MurMur algorithm
excerpt_separator: <!--more-->
author: Miha J.
tags: powershell
---

Here is a function of

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