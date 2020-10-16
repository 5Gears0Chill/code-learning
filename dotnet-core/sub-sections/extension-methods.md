# Useful Extension Methods

The following documentation provides useful extension methods when creating a DOTNET CORE project.

## Paged List Extension

Pagination is a part of any API. In order to enforce a simple coding pattern around pagination a useful extension method may be written:

```c#
public static class PagedListExtension
{
    public static IQueryable<T> PageQuerable<T>(this IQueryable<T> query, int page, int pageSize)
    {
        int skip = Math.Max(pageSize * (page - 1), 0);
        return query.Skip(skip).Take(pageSize);
    }

    public static IEnumerable<T> PageEnumerable<T>(this IEnumerable<T> query, int page, int pageSize)
    {
        int skip = Math.Max(pageSize * (page - 1), 0);
        return query.Skip(skip).Take(pageSize);
    }
}
```

The above methods allow for a simple method attachment to page either an `IQueryable` (for DB queries) or an `IEnumerable` for lists that are already in memory.

---

## Distinct By Extension 

Removing duplicates from a list of complex objects is no simple task. the `System.Linq` namespace provides a `Distinct` method which allows for removing duplicates from an `IEnumerable` of primitive types where it be `string` , `int` or any other primitive type. However imagine having an object as follows:

```c#
public class User
{
	public int Id { get; set; }
    public string Name { get; set; }
}
```

The goal is to filter out any `User` who has the same name. Introducing the `DistinctBy` Extension:

```c#
 public static class DistrinctByExtension
 {
     public static IEnumerable<TSource> DistinctBy<TSource, TKey>
         (this IEnumerable<TSource> source, Func<TSource, TKey> keySelector)
     {
         HashSet<TKey> seenKeys = new HashSet<TKey>();
         foreach (TSource element in source)
         {
             if (seenKeys.Add(keySelector(element)))
             {
                 yield return element;
             }
         }
     }
 }
```

Using a `HashSet` we have the ability to `yield return` elements that have been seen. A `HashSet` by definition will only hold unique types. Therefore the above method may be used in the following way:

```c#
var filtered = users.DistinctBy(x => x.Name);
```

---

## Is Null and Is Not Null Extensions

The `if(foo == null)` is an expression written too often in code. Hence the following extension methods:

```c#
public static bool IsNull(this object obj)
{
	return obj == null;
}

public static bool IsNotNull(this object obj)
{
	return !obj.IsNull();
}
```

Simply attaching this onto any `object` allows you to write neater `if` statements such as `if(foo.IsNull())`

---

## Contains Any Extension

String extensions prove to be useful in any scenario. Contains is an extension method provided by Microsoft to determine if a substring can be found in a parent string. Contains Any is a custom extension method that allows for an `IEnumerable` of substrings to be passed in for comparison.

```c#
public static bool ContainsAny(this string input, IEnumerable<string> containsKeywords, StringComparison comparisonType)
{
 	return containsKeywords.Any(keyword => input.IndexOf(keyword, comparisonType) >= 0);
}
```

---

## To Camel Case Extension

Takes the first letter of a string and sets it to its lower format. Really... That's it... Nothing special here

```c#
 public static string ToCamelCase(this string str)
 {
     if (String.IsNullOrEmpty(str) || Char.IsLower(str, 0))
         return str;

     return Char.ToLowerInvariant(str[0]) + str.Substring(1);
 }
```

---

## Conditional Where Extension

To avoid extensive filtering with `String.IsNullOrEmpty(myString)` we create an extension method to simplify this 
```c#
public static IQueryable<T> ConditionalWhere<T>(this IQueryable<T> source, Func<bool> condition, Expression<Func<T, bool>> predicate)
{
    if (condition())
    {
        return source.Where(predicate);
    }
    return source;
}
```
Which can be used like this ` .ConditionalWhere(() => request.Name.IsNotNull(), x => x.Name == request.Name)` when filtering by a `Name` that may or may not be null

---

## Change Log

- [01-10-2020] - Added Paged List Extension
- [01-10-2020] - Added Distinct By Extension
- [01-10-2020] - Added Is Null and Is Not Null Extensions
- [02-10-2020] - Added Contains Any Extension
- [16-10-2020] - Added To Camel Case Extension
- [16-10-2020] - Added Conditional Where