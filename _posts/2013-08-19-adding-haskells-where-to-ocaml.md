---
layout: post
title: Adding Haskell's `where` keyword to OCaml
permalink: adding-haskells-where-to-ocaml.html
---


I've recently been learning OCaml in my free time at the [Hacker School](http://www.hackerschool.com/) space.

Normally, if you want to define multiple variables in OCaml, you chain a bunch of `let`s together. This works in roughly the same way lisp's `let` and `let*` work:

```ocaml
(* OCaml function, computes (x+2)*2 *)
let foo x = let y = x+2 in
            let z = y*2 in
            z
```

Gross. That returns `z` if you didn't catch it---it's not very readable.

In Haskell, there is a convenient solution to this problem: the `where` keyword.

```haskell
-- if we were writing this in Hasekell:
foo x = z
  where y = x+2
        z = y*2
```

Way better. We can clearly see we're returning `z`, and it is obvious how we computed `z`.

Unfortunately, while this is idiomatic in Haskell, vanilla OCaml does not support `where`.

Because I miss this feature greatly, I decided to extend OCaml to support this feature.

The core control logic of this patch is about 6 lines of code, but the total patch is about 30 lines of code, due to scaffolding. (**Full source available [here](https://github.com/hausdorff/ocaml-where), language extension code specifically is [here](https://github.com/hausdorff/ocaml-where/blob/master/where.ml)**.) Practically speaking, even if you don't know that much about OCaml or Haskell, you can still understand this!

Here I'll walk through the process of building such extension.

## OCaml Syntax extensions, the 10,000-foot view

Extensions to OCaml core syntax are added in a step that happens just before compilation. It is called *preprocessing*.

Preprocessing works a lot like lisp's macro system. We take as input some OCaml source with our new modified syntax (*e.g.*, the `where` keyword), and transform it into "regular" OCaml. From here, it gets passed to the regular OCaml compiler, and turned into a real binary.

So, for example, say you wrote this silly program using `where` syntax.

```ocaml
(* new OCaml `where` syntax *)
let x = z
  where y = 10 and
        z = 20
```

*(Aside: you may have noticed that our `where` syntax includes an `and` separating the `y` case and the `z` case. This is a difference between our OCaml syntax and the Haskell syntax, and it exists for technical reasons. Disregard it for now, we will see why this is true in a minute.)*

When we pass this source off to the build system, it gets magically turned into the equivalent but uglier program below, which obeys "normal" OCaml syntax rules.

```ocaml
(* transformed to "regular" OCaml syntax *)
let x =
  let y = 10 in
  let z = 20 in
  z
```

We turn the pretty program into this ugly version only so that the OCaml compiler can turn it into a binary. But the user never sees this modified program---all they see is the code they wrote with `where`, and then a binary that just works. Because this transformation step is invisible, it seems like OCaml itself supports `where`!

This leaves the question of how to actually perform this transformation.

At a high level, we want to:

1. augment the OCaml parser to recognize this new `where` syntax in source code, and
2. upon finding an example of such syntax, simply transforming it directly into the equivalent set of `let`s

As we will see, the complete control logic for both of these steps is about 6 lines of code. Not so complicated! The rest of the ~30 lines of our patch file is just scaffolding.

## Getting familiar with the OCaml grammar

Augmenting the OCaml grammar generally begins by looking at the OCaml parser (*e.g.*, [v3.12.1](https://github.com/diml/ocaml-3.12.1-print/blob/master/camlp4/Camlp4Parsers/Camlp4OCamlParser.ml)), which specifies the grammar. This is because adding a syntax extension involves inserting new rules into the OCaml grammar.

Note that the official OCaml parser is implemented in a so-called *revised* version of OCaml (source [here](https://github.com/diml/ocaml-3.12.1-print/blob/master/camlp4/Camlp4Parsers/Camlp4OCamlRevisedParser.ml)). So, while the revised OCaml reads a lot like regular OCaml, just note it's actually a bit more than vanilla OCaml.

Here's where poking around the source actually pays off for us.

If you actually look through the revised parser, you'll eventually see that it does implement a weakened version of `where` (located [here](https://github.com/diml/ocaml-3.12.1-print/blob/master/camlp4/Camlp4Parsers/Camlp4OCamlRevisedParser.ml#L613)). Don't worry about what this code means just yet---we'll go through it in a bit:

```ocaml
    expr:
      [ "top" RIGHTA
<... omitted code here ...>
      | "where"
        [ e = SELF; "where"; rf = opt_rec; lb = let_binding ->
            <:expr< let $rec:rf$ $lb$ in $e$ >> ]
```

But then, in the official OCaml parser, at the bottom of the file ([here](https://github.com/diml/ocaml-3.12.1-print/blob/master/camlp4/Camlp4Parsers/Camlp4OCamlParser.ml#L137)) the `where` keyword is actually *deleted*! So it has the rule, and then removes the rule.

```ocaml
DELETE_RULE Gram expr: SELF; "where"; opt_rec; let_binding END;
```

Who knows why, but we can thank them because their version is not great, and it gives us the opportunity to do it right! :)

Overall, this is a pretty big win for us. It tells us where in the grammar we want to put our `where` rule, and it gives us a nice starting implementation which initially kind of sucks, but which we can modify to suit our needs.

But the question remains: *how* do we implement this?

## Overview of `Camlp4` grammar extensions

*(NOTE: If you don't know a lot about parsing or abstract syntax trees, it may be beneficial to have a look at some other resources, like [my article on parsing Python](/obvious-python-parser.html). We can't fit everything in this article!.)*

The OCaml preprocessing step is usually done by a utility like `Camlp4` or `Camlp5`. The basic job of these utilities is to parse a bunch of OCaml source, and hand off the abstract syntax tree (AST) to the OCaml compiler, which will eventually turn it into a binary.

Both `Camlp4` and `Camlp5` are woefully underdocumented. Personally, I had better luck with `Camlp4`, so that's what we'll use to extend OCaml here.

For the rest of this section, our objectives are to:

1. fix the `where` rule so that it doesn't suck anymore, and
2. glue this new rule into the OCaml grammar.

Let's have a look at how `Camlp4` expects such extensions to be formatted.

```
entryname: [ "levelname"
             [ pattern -> what_to_do_once_we've_found_pattern ]
           | "anotherlevelname"
             [ another_pattern -> etc ]
           ];
```

Collectively, this is known as a `Camlp4` *entry*. An entry, very vaguely, contains a list of patterns to find (specified as [context-free grammars](http://en.wikipedia.org/wiki/Context-free_grammar) aka CFGs), and tells us what to do with them once we've found them.

* Entries have *names*, like `entryname`. These are like function names---they exist so that other entries can refer to the pattern by name, making it easy to reuse old patterns.
* The entry itself---everything between the name of the entry (in this case, `entryname`) and the semicolon at the end of the code segment. Think of this as a list of *levels*---we'll see in a minute what levels are.
* An entry consists of a bunch of *levels*. This example contains two levels: `[ pattern -> what_to_do_once_we've_found_pattern ]` and `[ another_pattern -> etc ]`. Each level can have a name---the name of our first level is `"levelname"`. Each level contains a pattern, like `pattern`, which are written as CFGs. We can define functions like `what_to_do_once_we've_found_pattern` that tell us what to do once we've found a pattern. In general, you can group levels under the same entry to conveniently do things like make operator precedence. We don't care that much about that part of levels here, but if you're interested, there is a more detailed description [here](http://ambassadortothecomputers.blogspot.co.uk/2010/05/reading-camlp4-part-6-parsing.html).
* An entry can contain an optional *insertion position*, which tells us where in the grammar to put this new rule. This rule doesn't have an insertion position, but we'll make one when we modify it. Stay tuned.

Armed with this knowledge, we're ready to try to understand the old version of `where`. Let's have a look:

```ocaml
expr: 
  [ "where"
        [ e = SELF; "where"; rf = opt_rec; lb = let_binding ->
            <:expr< let $rec:rf$ $lb$ in $e$ >> ]
  ];
```

So it's an entry called `expr`, which contains one level called `"where"`.

The pattern in this case is: `e = SELF; "where"; rf = opt_rec; lb = let_binding`. The first part, `e = SELF`, is just a recursive call to `expr`, stored in the variable `e`---they could have just written `e = expr`, but for some reason they didn't. We store the result in the variable `e` because we want to use the result later.

The second part matches the string literal `"where"`. So, that means that so far we're looking for a pattern like "SOME_EXPRESSION `where` *blah*".

The expression `rf = opt_rec` calls another rule called `opt_rec` that optionally has the keyword `rec` inside it.

Finally, the expression `lb = let_binding` calls another rule called `let_binding` that matches to the part of `let` statements that binds values to variables. So, in an expression like `let x = 10 in x+2`, the `let_binding` is `x = 10`.

At this point we understand the pattern, but what do we do with it once we find the pattern in some OCaml code? That is handled by the expression `<:expr< let $rec:rf$ $lb$ in $e$ >>`.

Let's break this down. (This will be easier if you know about lisp quotations.) First, notice that in OCaml quotations have the pattern `<:rule_handler< OCAML_CODE >>`. That is, `OCAML_CODE` is surrounded by a quotation open tag (`<:rule_handler<`) and a quotation close tag (`>>`). Never mind what `rule_handler` is for now.

In our old `where` entry, `OCAML_CODE` would be `let $rec:rf$ $lb$ in $e$`. But notice that `rf`, `lb`, and `e` are all variables that were defined in the parsing pattern!

This is important because this code is basically taking those variables and splicing them into a `let` expression. More specifically, if you write `let $lb$ ...` and `lb` is an expression like `x = 10`, then the result is `let x = 10 ...`, since the `$`s tell `Camlp4` that we want to splice the value of `lb` into the current expression literal.

So, to complete the example, if we wrote `let x = z where z = 10`, then `rf` would be empty (since there's no `rec` keyword here), `lb` would be `z = 10`, and `e` would be the expression `z`. Thus, due to splicing, the new `let` statement would be `let x = let z = 10 in z`. Confusing, ugly, but equivalent.

This leaves one question. What does the quoting actually do? What is `rule_handler`? The abbreviated explanation is that `<:rule_handler< OCAML_CODE >>` will take some newly-spliced chunk of `OCAML_CODE` and expand it out to an AST in preparation for handing it off to the OCaml compiler. How it is expanded is determined by the handler `rule_handler`---someone registers a function to call to do the expansion, and it is called when we see the pattern. In the end, though, the effect is to take some new bit of syntax (in our case, the `where` keyword), rewrite it as "standard" OCaml, and then hand that to the compiler, which will turn it into an executable.

At this point we understand pretty much everything about the old rule, and we're ready to fix it up to be more like Haskell's `where`.

## Fixing up the old `where`

The problem with the old `where` is that we are only allowed to have one expression. In other words, the following is allowed:

```ocaml
let x = z
  where z = 20
```

But, on the other hand, this other example is not allowed, because it has two binding expressions:

```ocaml
let x = z
  where y = 10 and
        z = 20
```

So our task is to extend the above rule to account for this. My solution the following (see full source [here](https://github.com/hausdorff/ocaml-where/blob/master/where.ml)):

```ocaml
let_binding_seq: [[ rf = opt_rec; lb = let_binding -> (rf,lb) ]];
expr: BEFORE ":="
  [ "where"
    [ e = SELF; "where"; lb = LIST1 let_binding_seq SEP "and" ->
      <:expr< $exp_let_bindings e lb$ >> ]
  ];
```

There are only a couple changes here. The first is to add an entry, `let_binding_seq` which basically matches a single `let_binding`, and an optional keyword, `rec`. It emits them as a tuple, which makes it convenient to splice the parse results together later.

Inside the `expr` entry, we match the `where` keyword, and then following that is a list of one or more `let_binding_seq`s, each separated by the keyword `and`---this is denoted as `LIST1 let_binding_seq SEP "and"`.

Important note: this syntax diverges somewhat from the Haskell syntax. In Haskell, we'd write something like this:

```haskell
foo x = z
  where y = x+2
        z = y*2
```

In our extension, we'd have to put an `and` between the two bindings:

```ocaml
let foo x = z
  where y = x+2 and
        z = y*2
```

The reason for this is that one of the entries that `let_binding` depends on (specifically, `fun_binding`, as we see [here](https://github.com/diml/ocaml-3.12.1-print/blob/master/camlp4/Camlp4Parsers/Camlp4OCamlRevisedParser.ml#L779)) is right-associative and would group the above as `let y = (x+2; z = y*2)`, which is not what we want. By interspersing `and` between the bindings, we force the parser to break them up correctly.

Notice also that after declaring `expr` we see `BEFORE ":="`. This is the location rule we talked about earlier---it basically puts this entry before the entry for `:=`. We want this because in the original grammar, [you can see](https://github.com/diml/ocaml-3.12.1-print/blob/master/camlp4/Camlp4Parsers/Camlp4OCamlRevisedParser.ml#L616) that `where` appears just before the entry for `:=`.

The last major difference is how we splice in the list of bindings into the chained `let`s. To do this, we define another function:

```ocaml
let rec exp_let_bindings body = function
  | [] -> <:expr< >>
  | [(rf,lb)] -> <:expr< let $rec:rf$ $lb$ in $body$ >>
  | (rf,lb)::xs -> <:expr< let $rec:rf$ $lb$ in $exp_let_bindings body xs$ >>
```

This recursively builds the chain of `let`s. The base case is identical to the original rule; in every other case, we simply recurse build a `let` for the current binding and recurse into the list. Note that the empty case will definitely result in a parse error.

## Miscelleny and hooking it all into OCaml

The rest of the file is pretty much all scaffolding. Let's look at the whole file:

```ocaml
open Camlp4

module Id : Sig.Id = struct 
  let name = "where" 
  let version = "0.1" 
end 

module Make (Syntax : Sig.Camlp4Syntax) = struct
  include Syntax

  let _loc = Loc.ghost
  let rec exp_let_bindings body = function
    | [] -> <:expr< >>
    | [(rf,lb)] -> <:expr< let $rec:rf$ $lb$ in $body$ >>
    | (rf,lb)::xs -> <:expr< let $rec:rf$ $lb$ in $exp_let_bindings body xs$ >>

    EXTEND Gram
      GLOBAL: expr;

      let_binding_seq: [[ rf = opt_rec; lb = let_binding -> (rf,lb) ]];
      expr: BEFORE ":="
        [ "where"
          [ e = SELF; "where"; lb = LIST1 let_binding_seq SEP "and" ->
            <:expr< $exp_let_bindings e lb$ >> ]
        ];
    END
end
  
module M = Register.OCamlSyntaxExtension(Id)(Make)
```

Let's look at a few things that are important to understand.

The line `module M = Register.OCamlSyntaxExtension(Id)(Make)` registers our extension; if you don't do this, it isn't incorporated into the parser. It takes as arguments two modules. `Id` is defined at the top of the code and is used to identify what it is, containing the name and the version number. `Make` is the module that contains our actual syntax extension.

The line `let _loc = Loc.ghost` annotates code with the location we encountered it, *e.g.*, the file, line number, and character offset. `Loc.ghost` is a dummy value indicating no location. This makes sense because this is a macro, which doesn't exist in the source code of the project we'recompiling. This line exists as an important technicality: the quote expansion uses a variable `_loc`, which means you have to define it so that it's in scope during quote expansion. It's really annyoing and a bad decision point, but it's necessary.

The line `EXTEND Gram` indicates that the following entries extend the standard OCaml grammar. `Gram` is the module that controls this.

It is probably obvious that the line `END` indicates that we're done `EXTEND`'ing `Gram`.

The line `GLOBAL: expr;` is a hack that allows us to skip global declaration of the entry `let_binding_seq`. You have to globally declare each entry not specified in this way, and we'd rather not do that for a variety of reasons. Note that `expr` already exists in the OCaml grammar, while `let_binding_seq` only exists locally.


## Using it in real projects!

In the makefile, you can see that I compile the souce with the command `ocamlc -pp "camlp4o pa_extend.cmo q_MLast.cmo" -I +camlp4 -c where.ml`.

This results in `where.cmo`.

At this point, you can link it into whatever project you have, *e.g.*, `camlc -pp "ocamlp4o ./where.cmo" your_ml_file_here.ml`.

To see the desugared version of your source code, you can run `camlp4o where.cmo your_ml_file_here.ml`.

The only issue is that it might not be usable with other extensions, but that is sort of the way these things go.


## Wrapping it all up

So there you go, a shiny new `where` for all you OCaml fans.

The most challenging part of this journey was the incredibly sparse documentation, so I'll list some resources I found helpful for learning how to do this.

* Jake Donham's [excellent tutorials on `Camlp4`](http://ambassadortothecomputers.blogspot.co.uk/p/reading-camlp4.html). Walks from quotations to full syntax extensions. I ended up using `Camlp4` just because this is so good that I learned `Camlp4`, and the documentation was so bad (and out of date) I couldn't learn anything else. Specifically helpful were the chapters on [parsing](http://ambassadortothecomputers.blogspot.co.uk/2010/05/reading-camlp4-part-6-parsing.html) and [syntax extensions](http://ambassadortothecomputers.blogspot.co.uk/2010/09/reading-camlp4-part-11-syntax.html).
* Souce for the [Camlp4 OCaml parser](https://github.com/diml/ocaml-3.12.1-print/blob/master/camlp4/Camlp4Parsers/Camlp4OCamlParser.ml) and the [Camlp4 OCaml revised parser](https://github.com/diml/ocaml-3.12.1-print/blob/master/camlp4/Camlp4Parsers/Camlp4OCamlRevisedParser.ml). This is mirrored by [Jérémie Dimino](https://github.com/diml), and saved me a ton of time.
* The [`Camlp4` changelist](http://nicolaspouillard.fr/camlp4-changes.html), which helped me solve compatibility issues.
* The slight-awful [`Camlp4` project page](http://brion.inria.fr/gallium/index.php/Camlp4). Specifically the parts on [syntax extension](http://brion.inria.fr/gallium/index.php/Syntax_extension) and [quotation](http://brion.inria.fr/gallium/index.php/Quotation).
* The calmunity source mirror ([*e.g.*](http://camlunity.ru/doc/camlp4-3.12/Camlp4.Sig.Camlp4Ast.html)), which helped me navigate many type errors. Painfully navigate.
* Begrudgingly, I acknowledge the woefully out-of-date official [`Camlp4` tutorials](http://caml.inria.fr/pub/docs/tutorial-camlp4/). Sometimes you can follow along until something is wrong and still get something out of it. Often contains broken English in addition to broken code. I looked heavily at [lambdas](http://caml.inria.fr/pub/docs/tutorial-camlp4/tutorial004.html), [tutorial 7](http://caml.inria.fr/pub/docs/tutorial-camlp4/tutorial007.html), [tutorial 3](http://caml.inria.fr/pub/docs/tutorial-camlp4/tutorial003.html)
* There was a [denser tutorial](http://mjambon.com/extend-ocaml-syntax.html) that I didn't get around to looking at.



<p></p><br/>
<p></p><br/>

*If you enjoyed this post, you should consider applying to [Hacker School](https://www.hackerschool.com/), which is where this work was done. See some [other stuff](/obvious-python-parser.html) I did at Hacker School for more reference points. The space is full of some of the [nicest, smartest hackers](https://www.hackerschool.com/residents) I have ever met. Working around them has been a joy and an inspiration.*







