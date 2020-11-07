---
layout: post
title: Skip xUnit tests in a GitHub actions pipeline
excerpt_separator: <!--more-->
author: Miha J.
tags: github xunit
---

We use `GitHub` and `Azure DevOps` agents and runners to run pipelines to make our development processes more robust and automated.

Of course, agents have some boundaries to what they can do. For example, running integration tests that need to start a process on the agents usually are not allowed (shared agents). We are not allowed primarily for security reasons, and that is fine.

To avoid this, you can run your agent or cheat and skip those tests entirely in a specific environment.

Let me show you how to do that. In your Test project, create a new class:

```csharp
using System;
using Xunit;

public sealed class IgnoreOnGithubFact : FactAttribute
{
    public IgnoreOnGithubFact()
    {
        if (IsGitHubAction())
        {
            Skip = "Ignore the test when run in Github agent.";
        }
    }

    private static bool IsGitHubAction()
        => Environment.GetEnvironmentVariable("GITHUB_ACTION") != null;
}
```

When the agent runs in the GitHub environment, it contains many default Environment variables we can access. In the code above, we check that the `GITHUB_ACTION` variable is not null, and if it is, we skip the test. Use it on your tests like this:

```csharp
[IgnoreOnGithubFact]
public void MyTest()
{
    var sut = new MyClass("123");

    Assert.Equal("123", sut.GetValue());
}
```

Those tests will not run next time you run the test with the GitHub agent. :)