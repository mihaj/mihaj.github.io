---
layout: post
title: Invalid string format in model validation message in .NET
excerpt_separator: <!--more-->
author: Miha J.
tags: net6, net7, c#, data annotations, invalid string format exception
---

I recently encountered a strange case or runtime error in `System.ComponentModel.DataAnnotations` namespace.

I have a model below and, with it, a few model validation attributes.

```c#
public class MyModel
{
    [Required]
    [MinLength(1)]
    [MaxLength(64)]
    [RegularExpression("^[0-9\\p{L}\\-_,.!?\\[\\]{}()<> ]*$", ErrorMessage = "is invalid. It should only contain letters, numbers, spaces, unicode and special characters ( ) [ ] { } < > - _ , . ? !")]
    [NoWhitespaceOnStartAndEnd(ErrorMessage = "is invalid. It should not start or end with space character(s).")]
    public string Name { get; set; }
}
```

Nothing weird on the first look. The code also compiles fine. But when I try to validate the property `Name` with some strange inputs, I get the exception below:

```c#
System.FormatException: Input string was not in a correct format.
   at System.Text.ValueStringBuilder.ThrowFormatInvalidString()
   at System.Text.ValueStringBuilder.AppendFormatHelper(IFormatProvider provider, String format, ReadOnlySpan`1 args)
   ...
```

This does not look like an error of the validator but something internally. After some digging, I discovered that `ErrorMessage` was the culprit.

After some digging in .NET internals, I found this:

```c#
public override string FormatErrorMessage(string name)
{
    SetupRegex();

    return string.Format(CultureInfo.CurrentCulture, ErrorMessageString, name, Pattern);
}
```

That makes sense the ErrorMessage string is formatted using `string.Format()`. I isolated the case and ran the code below a few times until I tried curly braces. I got the same error:

```c#
var str = string.Format("Miha {}");
> System.FormatException: Input string was not in a correct format.
  + System.Text.StringBuilder.FormatError()
  + System.Text.StringBuilder.AppendFormatHelper(System.IFormatProvider, string, System.ParamsArray)
  + string.FormatHelper(System.IFormatProvider, string, System.ParamsArray)
  + string.Format(string, object[])
```

Now let's escape them:

```angular2html
> var str = string.Format("Miha {{}}");
> Console.WriteLine(str);
Result: Miha {}
```

Now I only need to escape curly braces with double curly braces in my model validation error message to:

`[RegularExpression("^[0-9\\p{L}\\-_,.!?\\[\\]{}()<> ]*$", ErrorMessage = "is invalid. It should only contain letters, numbers, spaces, unicode and special characters ( ) [ ] {{ }} < > - _ , . ? !")]`
And that's it! :D