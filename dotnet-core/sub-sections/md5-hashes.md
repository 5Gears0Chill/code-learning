# Generating MD5 Hashes 

Microsoft provides an extensive cryptography namespace in which multiple methodologies are defined. On of which allows for the generation of an MD5 Hash.

A quick explanation of MD5 is as follows 

> The **MD5 message-digest algorithm** is a widely used [hash function](https://en.wikipedia.org/wiki/Hash_function) producing a 128-[bit](https://en.wikipedia.org/wiki/Bit) hash value 

Below is the methodology for generating an MD5:

```c#
 private static string GenerateMD5Hash(string input)
 {
     var md5 = new MD5CryptoServiceProvider();
     var bytes = Encoding.UTF8.GetBytes(input);
     bytes = md5.ComputeHash(bytes);
     var sb = new StringBuilder();
     foreach (byte b in bytes)
     {
         sb.Append(b.ToString("x2").ToLower());
     }
     return sb.ToString();
 }
```

An Explanation of `ToString("x2")` is as follows:

> It formats the string as two uppercase hexadecimal characters.
>
> In more depth, the argument `"X2"` is a "format string" that tells the `ToString()` method how it should format the string. In this case, "X2" indicates the string should be formatted in Hexadecimal.
>
> `byte.ToString()` without any arguments returns the number in its natural decimal representation, with no padding.
>
> Microsoft documents the [standard numeric format strings](http://msdn.microsoft.com/en-us/library/dwhawy9k(v=vs.110).aspx) which generally work with all primitive numeric types' `ToString()` methods. This same pattern is used for other types as well: for example, [standard date/time format strings](https://docs.microsoft.com/en-us/dotnet/standard/base-types/standard-date-and-time-format-strings) can be used with `DateTime.ToString()`. - [StackOverFlow](https://stackoverflow.com/questions/20750062/what-is-the-meaning-of-tostringx2)

