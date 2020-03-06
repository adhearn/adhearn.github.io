---
layout: post
title: Day 2: Progress Update and Interesting/Confusing Things I Learned Today
date: 2020-03-05
categories:
published: false
---
## Day 2 Progress Update
Finished Chapter 2 of the Dragon Book. As expected, once I passed lexing and parsing I was on much more familiar ground and moved a lot faster. C continues to be a challenge - things I can do effortlessly in Python or Java require research and debugging, but I'm starting to feel the rust fall away.

## Using Debuggers
I have a bad habit of print debugging instead of using debuggers. That actually works pretty well when using tools like Jupyter notebooks to write Python. With C, I've found it's generally faster and easier to jump into gdb than it is to add print statements to the code, reompile, and re-run, especially since I usually end up needing to iterate on that process multiple times. It's like the activation energy for a chemical reaction has been lowered, making it net lazier to use gdb instead of printf. I'm hoping my newfound respect for using the debugger carries over to the next time I use Python.

## Confusing Sentence About ASTs from the Dragon Book
I came across a sentence in the section on Intermediate Code Generation in Chapter 2 (section 2.8.1) that I read several times without getting any real clarity on: `However, it is common for compilers to emite the three-address code while the parser "goes through the motions" of constructing a syntax tree, without actually constructing the complete tree data structure. Rather, the compiler stores nodes and their attributes needed for semantic checking or other purposes, along with the data structure used for parsing.`. My confusion is specifically around saying that the compiler stores nodes and teh data structure used for parsing, but not the AST. How are the nodes stored, if not in a tree? Just as entries in the symbol table? And what is the `data structure used for parsing` if not the AST? Do they just mean the parse tree/concrete syntax tree? If so, why bother storing that? As discussed in my previous post, that just contains some superfluous information that isn't relevant to the actual semantic constructs of the language. It seems like what you want out of the parser is the AST, not the parse tree. Anyway, I notice that I'm confused about this sentence. The authors say they will `take up the details of this process in Chapter 5`, so hopefully clarity is incoming.

## One Symbol Table Per Block
The authors of the Dragon Book provide a handy rule for constructing symbol trees for block-scoped languages: have one symbol table per block. When entering a new block, give a pointer to the enclosing symbol table. Simple enough.

What I found interesting was that I had run into a version of the problem this rule attempts to solve while messing around writing a Scheme interpreter in Python last week. Initially, I tried to implement variable lookup environments as Python dictionaries. However, this fails in the following case:

``` scheme
(let ([x 23])
  ((lambda (x) (+ x 1)) 42)
  x)
```

In my interpreter, the outer `let` would create a new entry to the environment, which looked like `{'x': 23}`. Then, when the lambda was applied, we would extend the environment to set `x` to `42` before evaluating the lambda body, `(+ x 1)` in this case. However I had naively implemented environment extension so that the old value of x was overwritten, so our environment looked like `{'x': 42}`. Then, when the final `x` was evaluated, it was evaluated as `42`, not `23`. My gut instinct was that we needed to remove any newly introduced variables from the environment after they are out of context, but that obviously fails because then the environment has no record of `x` at all! Once I realized what the real problem was, I quickly came up with a solution that involved copying the dictionaries on environment extension. However, a better solution would have been to use the "one symbol table per scope" rule to implement environments. The underlying storage could have continued to be in a dictionary, but with a pointer to the previous environment.

Incidentally, this was only not a problem in Scheme because the most obvious representation for an environment in Scheme is an association list, i.e. a linked list of pairs. The first element in each pair is the variable name, the second element is the variable's value. When entering a new scope, you simply keep a pointer to the old environment around, process the new scope with the extended environment, then use the pointer you saved as the environment when processing things outside of the new scope. You could do something similar in Python by tracking the length of the environment when entering the new scope, then slicing any newly introduced variables, but then you would have to do environment lookups from the back of the list to handle variable shadowing. All in all, the one environment per scope implementation seems nicer.
