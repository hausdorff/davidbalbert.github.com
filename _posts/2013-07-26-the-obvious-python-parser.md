---
layout: post
title: The obvious Python parser
permalink: obvious-python-parser.html
---


**All parser code is available [here](https://github.com/hausdorff/pyli/blob/master/src/Parser.hs).**

I spent my first few weeks at [Hacker School](https://www.hackerschool.com/) writing a Python compiler from basically scratch.

The task of merely parsing a complete language like Python can be quite intimidating at the outset. I've found that many people simply assume it's nearly impossible.

I began to wonder if it was possible to write a parser so clear that it would seem obvious how the parser worked.

Let's look at some examples of where I took this.

> *NOTE: The parser is written in Haskell, but P-L-E-A-S-E do not worry if you don't know Haskell. I've endeavored to make it readable to everyone. Be sure to email me at `clemmer.alexander@gmail.com` if you have questions.*
<p></p>


## Parsing a file

The following is a function called `fileInput` that merely parses a file. (See original source [here](https://github.com/hausdorff/pyli/blob/master/src/Parser.hs#L28).)

```haskell
fileInput = lines <~> endOfFile ==> emitProgram
  where lines     = lineCfg
        endOfFile = ter "ENDMARKER"
```

I've tried to get this to mirror the way I would explain a Python file in English. This is how I'd read it in my head:

> A `fileInput` is a series of `lines` concatenated with an `endOfFile` marker, `where` our `lines` are defined using a function `lineCfg`, and `endOfFile` is defined using some function `ter` that we don't understand yet. Once we parse a Python file, we call a function `emitProgram`, which tells us what to do with the parsed Python source code.

I hope so far this seems intuitive. Let's go through the code more concretely.

The `<~>` operator just means "concatenate". So, we can see that a file consists of `lines <~> endOfFile`, or in other words, a bunch of lines concatenated with an EOF marker.

So the whole expression `lines <~> endOfFile ==> emitProgram` is just saying, once we parse a bunch of lines and an EOF marker, we want to call the function `emitProgram`, which will tell us what to do once we've finished parsing the program. (It emits the program as an abstract syntax tree, but don't worry about that just yet.)

If you haven't grokked it yet, the `where` clause is basically telling us how `lines` and `endOfFile` are defined. Don't worry too much about them just yet. They're not super important, and anyway if you really wanted to know you'd just need to look in the source for those definitions, and they would look basically like this one did. So you can read it, no sweat.


## Parsing a function

If you don't know Python, here's an example of a Python function. It's called `cowfun`, it takes no arguments and returns the string `"cow"`.

```python
def cowfun ():
    return "cow"
```

The following code (original source [here](https://github.com/hausdorff/pyli/blob/master/src/Parser.hs#L42)) parses arbitrary function definitions.

```haskell
funcdef = def <~> id <~> parameters <~> colon <~> body ==> emitFuncdef
  where def   = ter "def"
        id    = ter "ID"
        colon = ter ":"
        body  = suite
```

In English I would read this as:

> A function is the keyword `def` concatenated with an `id` (in this case this is, `cowfun`, the name of our function), a list of `parameters`, a `colon`, and a `body`. When we parse a function, we will call `emitFuncdef`.

There's nothing really new in this example. `<~>` still means concatenation, and the components like `def` and `colon` are defined in the `where` block.

## Ok, parsing---what is it, and what do you get when you do it?

Ok, so we have a nicely-written parser, but what does this "parsing" thing do, anyway?

In short, the parsing makes sure the Python source represents a legal Python program. It is a *syntactic* check, in that it looks through the syntax of the source code to figure out if it is legal or not.

A compiler takes in raw Python source. Each file starts as a single string. The first job is to *tokenize* this string.

The idea behind tokenization is to reduce the Python code to the simplest possible representation of the source that still retains all of the meaning of the program---programs are broken down into keywords, ids, number literals, string literals.

So in the example below, we're sending the program `def cowfun(): return 'cow'` into the lexer (from [my lexing/parsing suite](https://github.com/hausdorff/pyli)) via standard in. In response, we receive a set of tokens as output.

```
> echo "def cowfun(): return 'cow'" | make pyle
(KEYWORD def)
(ID "cowfun")
(PUNCT "(")
(PUNCT ")")
(PUNCT ":")
(KEYWORD return)
(LIT "cow")
(NEWLINE)
(ENDMARKER)
```

The *only* reason to tokenize before parsing is because it greatly simplifies parsing.

For example, in this representation, we are given that the keyword `def` is followed by the id `cowfun`. This means the parser's job is just to figure out if that ordering makes sense. If we didn't write the lexer, the parser's job would be to both discover that `def` is a keyword, then that it's followed by an id `cowfun`, and *then*, finally, whether that makes sense.

Splitting the lexing and parsing thus makes parsing easier.

So what happens after parsing? We get back an *abstract syntax tree*, or AST.

Roughly speaking, you can think of a program as a tree---a `while` statement will have a predicate and a body, which can itself contain more `while` statements.

If we consider each expression or statement a node in a tree, then each thing in the body of the `while` is a child of the `while` statement.

Let's see that last example as an AST instead of a token stream (using the parser from my [my lexing/parsing suite](https://github.com/hausdorff/pyli)):

```scheme
> echo "def cowfun(): return 'cow'" | make pyli
'(program
  (def
   (cowfun)
   ((return
     "cow"))))
```

**This is the ultimate output of the parsing functions I showed you earlier. Just an AST.**

Here we see that `def` is a child of `program`. Similarly, the name `cowfun` and the `return` statement are children of `def`.

## Conclusions and next steps.

While I can't impart all of parsing Python onto you in this short post, I hope that I have convinced you that **you can understand the source code of this parser if you want to**.

It is not a black art.

It is simply a matter of reading the code and (maybe) asking questions. To me or the program.

For the future, I would propose the following.

* Poke around with [the source](https://github.com/hausdorff/pyli/blob/master/src/Parser.hs). You should now know most of what you need to know to understand the whole parser!
* Run it over various programs. Note that this parser actually doesn't cover all of Python. In particular, it doesn't cover tabs, decorators, bytestrings, and some other stuff.
* As an exercise, *add tab support, decorators, and bytestrings to the parser*.
* Above all, have fun when you do it!

<p></p><br/>
<p></p><br/>

*If you enjoyed this post, you should consider applying to [Hacker School](https://www.hackerschool.com/). The space is full of some of the [nicest, smartest hackers](https://www.hackerschool.com/residents) I have ever met, and whithout whose thoughtful feedback, this work would not have been done.*


<p></p><br/>
<p></p><br/>































