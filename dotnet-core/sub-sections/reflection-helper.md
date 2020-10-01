# Reflection Utilities in DOTNET CORE

In a scenario where objects need to be instantiated at runtime, DOTNET CORE has provided a name space `System.Runtime` which holds helper methods and classes to accommodate for reflective programming.

The following explanation will provide a useful way of creating object instances with any set of parameters into the constructor at runtime.

```c#
public delegate T ObjectActivator<T>(params object[] args);

public static class ObjectCreator
{
    public static TInterface NewInstance<TInterface>(Type objectType)
        => NewInstance<TInterface>(objectType, null);

    public static TInterface NewInstance<TInterface>(Type objectType, params object[] args)
    {
        ConstructorInfo ctor = objectType.GetConstructors().First();
        ObjectActivator<TInterface> createdActivator = GetActivator<TInterface>(ctor);
        return createdActivator(args);
    }

    public static TInterface NewInstance<TInterface, TObject>(params object[] args)
    {
        ConstructorInfo ctor = typeof(TObject).GetConstructors().First();
        ObjectActivator<TInterface> createdActivator = GetActivator<TInterface>(ctor);
        return createdActivator(args);
    }

    public static TObject NewInstance<TObject>(params object[] args)
    {
        ConstructorInfo ctor = typeof(TObject).GetConstructors().First();
        ObjectActivator<TObject> createdActivator = GetActivator<TObject>(ctor);
        return createdActivator(args);
    }

    private static ObjectActivator<T> GetActivator<T>(ConstructorInfo ctor)
    {
        Type type = ctor.DeclaringType;
        ParameterInfo[] paramsInfo = ctor.GetParameters();

        ParameterExpression param = Expression.Parameter(typeof(object[]), "args");

        Expression[] argsExp = new Expression[paramsInfo.Length];

        for (int i = 0; i < paramsInfo.Length; i++)
        {
            Expression index = Expression.Constant(i);
            Type paramType = paramsInfo[i].ParameterType;

            Expression paramAccessorExp = Expression.ArrayIndex(param, index);

            Expression paramCastExp = Expression.Convert(paramAccessorExp, paramType);

            argsExp[i] = paramCastExp;
        }

        NewExpression newExp = Expression.New(ctor, argsExp);

        LambdaExpression lambda = Expression.Lambda(typeof(ObjectActivator<T>), newExp, param);

        ObjectActivator<T> compiled = (ObjectActivator<T>)lambda.Compile();
        return compiled;
    }
}
```

---

##  Explanation of the code presented

The `ObjectCreator` static class provides a single method `NewInstance` with multiple overrides accommodating for multiple scenarios of constructor injection. Assume the following scenario:

`class A` requires to be instantiated at runtime but has the following signature

```c#
public class Foo 
{
	public Foo(int x)
	{
	}
    //any code for this class
}
```

The issue presented is that in order to instantiate `class A` it requires a value for `x`. With the proposed class above the following code is now possible.

```c#
int x = 5;
var foo = ObjectCreator.NewInstance<Foo>(typeof(Foo), new object[] { x });
```



---

## Change Log

- [01-10-2020] - Added Implementation of the Object Creator and an example usage case