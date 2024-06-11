---
layout: post
title: Implement IDistributedCache interface for Etcd key-value store
excerpt_separator: <!--more-->
author: Miha J.
tags: dotnet, etcd, idistributedcache
---

Applications sometimes need a temporary and faster cache storage
to fetch data from instead of going into a database and generate response.
With cache implemented, we can return just the response.
Using caches brings its own challenges to invalidate it, etc.

You are probably aware of `IDistributedCache` interface in AspNet framework
which is defining the methods for reading and writing the underlying cache infrastructure.
I've used Redis Cache in the past, but for the sake of exercise, I've implemented the [Etcd](https://etcd.io/) version of it.

Etcd is a distributed key-value store, and it's a perfect candidate for a cache store. Read more [here](https://etcd.io/).

Familiar registration of the service:
```csharp
builder.Services.AddEtcdCache(options =>
{
    options.ConnectionString = "http://localhost:2379";
    options.Username = "root";
    options.Password = "root";
});
```

Usage:
```csharp
public class MyClass
{
    private readonly IDistributedCache _distributedCache;

    public MyClass(IDistributedCache distributedCache)
    {
        _distributedCache = distributedCache;
    }

    public async Task Write(string key, string value)
    {
        //Save to cache
        await _distributedCache.SetAsync(key,
            Encoding.UTF8.GetBytes(value), new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
            });

        //Read from cache
        var cachedValue = await _distributedCache.GetAsync(key);
    }
    
    public async Task<string> Read(string key)
    {
        //Read from cache
        return await _distributedCache.GetAsync(key);
    }
}
```

You can check my first attempt to implement `IDistributedCache` for `Etcd` in my [GitHub repo](https://github.com/mihaj/EtcdCache). More work is probably needed to add other authentication type support to the configuration and clustering. I'll try to fill that in and create the `Nuget` as well.

Let me know what you think and any PR's are welcome :).