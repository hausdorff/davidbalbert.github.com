---
layout: post
title: "Avoiding dynamic dispatch in C# by \"manually\" JIT'ing the code yourself."
permalink: avoiding-dynamic-dispatch.html
comments: true
technique: ■
---


Let me ask you something. How many times in your life have you profiled your C# program and been like:

> Holy cow, dynamic dispatch is really killing my performance here.

Never.

"Never" is the answer. We're not Python programmers. We have a real JIT. We are not cavepeople.

Still, while "never" might be the acceptable, mostly-correct answer you'd find on StackOverflow, it turns out it's not the **fun** answer. There is no StackOverflow for **fun** answers.

The **fun** answer is: **We're going to manually JIT our code because we can. Fuck the police.**

The rest of this post will fall into four sections.

[**Section 1:**](#compiling) Programmatically compiling and running C# expressions at runtime.

[**Section 2:**](#dispatching) Compiling a call to a specific dispatch of a method.

[**Section 3:**](#under-the-hood) Under the hood. Where does this have perf impact?

[**Section 4:**](#production-lol) A hilarious time we actually used this in prduction.


<a name="compiling"></a>
## Programmatically compiling and running C# expressions at runtime.
I fucking love the [expression tree API](http://msdn.microsoft.com/en-us/library/bb397951.aspx).

If you don't know, the ET API allows you to symbolically construct, compile, and execute C# expressions at runtime.

To give you a taste of how this works, let's see an example. The following code snippet will (1) dynamically generate an expression representing a small lambda function, (2) compile that lambda function, and (3) run it.

```c#
Expression<Func<int, int>> exp = (x => x + 2);  // (1) create C# expression
Func<int, int> lambda = exp.Compile();          // (2) compile C# expression
int result = (int)lambda.DynamicInvoke(1);      // (3) run C# expression,
                                                //     passing in `1` as arg.
Debug.Assert(result == 3);
```

The first line mike look like normal lambda syntax, but it's actually an *expression literal*. The variable `exp` ends up being `Expression<int>` object, representing a lambda that adds two to the argument.

This expression literal syntax is provided only as a convenience. In some cases -- particularly when you're building really complicated C# expressions -- you will need to build up an expression tree without the expression literal syntax.

Since this is precisely what we'll have to do in the next section, let's redo the last example without the expression literal syntax, just to get a flavor for how this works.

(NOTE: **DO NOT WORRY** if you don't exactly get the subtleties of the code. What's important to understand is that **we're dynamically compiling and running C# expressions.** If you get the rest, that's gravy.)

```c#
ParameterExpression xParam = Expression.Parameter(     // (1) create the x parameter the
    typeof(int),                                       //     lambda expression uses
    "x");
LambdaExpression lambdaExpression = Expression.Lambda( // (2) create the lambda
    Expression.Add(xParam, Expression.Constant(2)),    //     expression itself
    xParam);
Delegate lambda = lambdaExpression.Compile();          // (3) compile
int result = (int)lambda.DynamicInvoke(1);             // (4) run
Debug.Assert(result == 3);
```

As you can see, this is a bit more opaque. Luckily, we will not need anything more complicated than this to manually JIT a method call.


<a name="dispatching"></a>
## "Manually" JIT'ing a method call with expression trees
Like Java, C# provides an API for *reflection*, so called because the API allows a running program to inspect itself, as it would if it had a fun little program mirror. Using this API, you can do things like get the runtime type of some object, or create an object whose type you don't know at compile time.

One of the facilities it provides is the ability to inspect individual methods. For example, in the following snippet, we obtain a method with the name "Add" that exists on the type `ICollection<int>`, and report the return type of that method. (In this case it would be `void`.)

```c#
MethodInfo interfaceAddMethod = typeof(ICollection<int>).GetMethod("Add");
var returnType = interfaceAddMethod.ReturnType;
```

The `MethodInfo` object here contains information specifically for the `Add` method. It's extremely useful in the context of expression trees, because it lets you compile expressions that call specific methods.

For example, in the following code snippet, we will dynamically get the `MethodInfo` for `List<int>.Add`, and then generate an expression that calls that method on some list, appending the number 4 to it. We then compile and run that expression.

```c#
var list = new List<int> { 1, 2, 3 };
var listAddMethod = typeof(List<int>).GetMethod("Add");
var addingExp = Expression.Lambda(
    Expression.Call(
        Expression.Constant(list),
        listAddMethod,
        Expression.Constant(4)));
var addingFunc = addingExp.Compile();
var result = addingFunc.DynamicInvoke();
``` 

Cool, eh? Now here's the payoff for your patience.

Have a look at these two `MethodInfo` objects.

```c#
var listAddMethod = typeof(List<int>).GetMethod("Add");
var interfaceAddMethod = typeof(ICollection<int>).GetMethod("Add");
```

These point at different methods.


<a name="under-the-hood"></a>
## Under the hood


<a name="production-lol"></a>
## So no one would ever do this, right?
Yeah, about that. We (Microsoft) totally exploit this in 

















