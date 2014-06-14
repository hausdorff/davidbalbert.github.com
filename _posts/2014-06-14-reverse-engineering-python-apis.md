---
layout: post
title: "Reverse engineering a Python API"
permalink: reverse-engineering-python-apis.html
comments: true
technique: ■
---


I'm sitting here on the 5 hour 40 minute flight from SEA to PHL. I need to use a regular expression to do something.

But, while normally I'd use the `re` module, I've forgetten the details of the API. And to top it off, I'm *certainly not* ponying up $14 for the crappy plane Internet!

I guess that means I have [*three* problems](http://regex.info/blog/2006-09-15/247).

Looks like I have to go in through the back door.


## What is the name of the method, again?

I start by cracking open my trusty Python interpreter and importing the standard regex module, `re`.

```
$ python
>>> import re
```

What methods are inside the module?

I honestly forget.

I remember I can call `dir(re)` to get "approximately" all the members of the `re` module.

I think `dir` is a function &mdash; it might be a metaclass or something, though. No Internet, remember?

Anyway, calling it results in:

```python
>>> dir(re)
['DEBUG', 'DOTALL', 'I', 'IGNORECASE', 'L', 'LOCALE', 'M', 'MULTILINE', 'S',
'Scanner', 'T', 'TEMPLATE', 'U', 'UNICODE', 'VERBOSE', 'X', '_MAXCACHE',
'__all__', '__builtins__', '__doc__', '__file__', '__name__', '__package__',
'__version__', '_alphanum', '_cache', '_cache_repl', '_compile',
'_compile_repl', '_expand', '_pattern_type', '_pickle', '_subx', 'compile',
'copy_reg', 'error', 'escape', 'findall', 'finditer', 'match', 'purge',
'search', 'split', 'sre_compile', 'sre_parse', 'sub', 'subn', 'sys',
'template']
```

Cool! Look at all those members! I wonder what they all do!

Looking closely, `re.match` and `re.findall` look familiar. That's a good start.


## How do I call this method, again?

Unfortunately, while those methods look familiar, I have no idea how to call or use them.

So, without docs, I need to figure out how to do that.

`dir` to the rescue again! This time we'll call `dir` with the method `re.findall` as a parameter:

```python
>>> dir(re.findall)
['__call__', '__class__', '__closure__', '__code__', '__defaults__',
'__delattr__', '__dict__', '__doc__', '__format__', '__get__',
'__getattribute__', '__globals__', '__hash__', '__init__', '__module__',
'__name__', '__new__', '__reduce__', '__reduce_ex__', '__repr__',
'__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'func_closure',
'func_code', 'func_defaults', 'func_dict', 'func_doc', 'func_globals',
'func_name']
```

WHOA! Look at those last few members! `re.findall.func_code`? `re.findall.func_doc`? WHAT EVEN ARE ALL THOSE THINGS, THEY LOOK AWESOME!

`re.findall.func_doc` seems like it might talk about the variables. Let's try it:

```
>>> print re.findall.func_doc
    Return a list of all non-overlapping matches in the string.

    If one or more groups are present in the pattern, return a
    list of groups; this will be a list of tuples if the pattern
    has more than one group.

    Empty matches are included in the result.
```

It doesn't! Damn. **And let that be a lesson to you. Document your parameters when you document your method.** Still, it does tell us a bit about the method. So, not completely useless.

What about `re.findall.func_code`?

```
>>> print re.findall.func_code
<code object findall at 0x105b36730, file "/Users/alex/PYTHON_STD/lib/python2.7/re.py", line 169>
```

What do we do with a "code object"? (You might say: the code object clearly lists a source file, it's on your machine. But it was 4 in the morning, and I ended up doing this later in the process, so hush up.)

`dir` to the rescue, yet again:

```python
>>> dir(re.findall.func_code)
['__class__', '__cmp__', '__delattr__', '__doc__', '__eq__', '__format__',
'__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__le__',
'__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__',
'__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'co_argcount',
'co_cellvars', 'co_code', 'co_consts', 'co_filename', 'co_firstlineno',
'co_flags', 'co_freevars', 'co_lnotab', 'co_name', 'co_names', 'co_nlocals',
'co_stacksize', 'co_varnames']
```

OOOOOOOOOOOOOOOoooooooo, `co_varnames` seems like it might be interesting!

```python
>>> re.findall.func_code.co_varnames
('pattern', 'string', 'flags')
```

Bingo. Those are the function arguments. The signature is `re.findall(pattern, string, flags)`


## So what does the `flags` parameter do?

Having discovered that `findall` has the signature `re.findall(pattern, string, flags)`, we now have to figure out what that last argument means.

I don't remember having used it before, so I suspect it's a default parameter.

When we look over the entries to `dir`, we see that in fact the default value for `flags` is 0.

```
>>> re.findall.func_defaults
(0,)
```

Can we confirm this is all true? Turns out we can get both the line and the file the source code is in:

```
>>> re.findall.func_code.co_filename
'/Users/alex/PYTHON_STD/lib/python2.7/re.py'
>>> re.findall.func_code.co_firstlineno
169
```

Going back to this source, we see:

```python
def findall(pattern, string, flags=0):
    """Return a list of all non-overlapping matches in the string.

    If one or more groups are present in the pattern, return a
    list of groups; this will be a list of tuples if the pattern
    has more than one group.

    Empty matches are included in the result."""
    return _compile(pattern, flags).findall(string)
```


## Did we even need `dir` at all?

Ok, no, we didn't. Your machine probably just has the source. But, I thought it would be fun to show off `dir` anyway. **It's still useful for the case when you have no documentation or code available at all!**

If you're up for more exploring, I suggest you keep poking at things with `dir`. The most interesting part is `re.findall.func_code`, which contains (among other things) members with the actual bytecode.