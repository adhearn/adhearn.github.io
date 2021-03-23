---
layout: post
title: "Post-Hiatus Plachutta Updates"
---

It's been a while! Between the holidays, work, applying to grad school, and the general blah-ness that comes over me most winters, I haven't been carving out enough time for personal projects. The PL itch is back, though, so I'm dusting off Plachutta. I had most of these thoughts written up before my hiatus, but reviewing them has been a nice to get back up to speed on what I was thinking about.

## Discovering QBE
I discovered [QBE](https://c9x.me/compile/), a compiler backend that’s much simpler than LLVM. If I had known it existed, I probably wouldn’t have bothered writing Plachutta in the first place - I would have just targeted QBE instead. Since Plachutta already exists, however, I’m probably going to make some changes that make it more similar to the intermediate language supported by QBE. That way, if I want to change targets I can do so with less fuss. Particularly, I stole the ideas for sigils and stack allocation instructions below from QBE, and will probably have more changes down the road to make Plachutta closer to a QBE IL interpreter.

## Fixes for Calls
The implementation for calls I suggested in “Adding Functions to Plachutta” doesn’t actually work - I failed to recognize that because calls appear on the RHS of assignments, the call visitor needs to return the value of the call, and can’t return the address to jump to the way jump instructions can. I got around this by a fairly atrocious hack. The first part of it is that we add two new instance variables to the interpreter, `return_value` and `return_value_set`. In the return visitor, we set `return_value_set` to True and place the return value in the `return_value` variable. In the call visitor, we first check to see if `return_value_set` is True. If it is, the value of the call is the value in `return_value_set`. Once we’ve read the value, we set `return_value_set` back to False and set `return_value` to None. (We can’t just use None or some other sentinel value because then that value can never be returned from a function.) If `return_value_set` is False, we manually set the `next_instruction` variable of the interpreter to be one less than the instruction pointed to by the label of the function we’re calling, set up a new StackFrame with a return address set to the address of the current instruction, and return None. The main eval loop increments the counter by one, and we jump to the beginning of the function we called with our new StackFrame ready to go. I dislike setting the `next_instruction` variable in an arbitrary visitor function, and I *really* dislike setting it to one less than the instruction we actually want. It binds the implementation of call too tightly to our specific eval loop. However, it gets the job done without messing around with multiple return values or other new instance variables.

An ever-so-slightly cleaner implementation might be to increment the `next_instruction` variable at the beginning of the eval loop, rather than the end. Then we can at least set it to the actual index of the label, rather than that index minus one. Incidentally, that also makes that name more correct. Currently, it actually points at the current instruction most of the time, rather than the next instruction. I should either change its name or change its value, before I confuse myself even more.

## Adding Command Line Arguments
Now that all Plachutta programs have to define a main function, it’s easy to support passing in command line arguments. The eval function can take a list of args, allocate space for them during setup of the initial stack frame, and pass two arguments to the main function: the length of the argument array, and the array itself. Easy enough.

## Using Sigils
One other solution to the global vs. local variable problem that I didn’t consider was using sigils to denote variable scope - basically, prefixing variable names with a scope identifier. Following [QBE’s usage of sigils](https://c9x.me/compile/doc/il.html#Sigils), I’m considering replacing global declarations with the following sigils:
- `$` for globals
- `%` for function-scoped variables
- `@` for labels

As an example, here’s the code I used to introduce the scope problem using sigils (with a minor extension to demonstrate globals):

```
$factcount = 0
@fact: $factcount = $factcount + 1
%x = param
if %x <= 1 jump @basecase
%sub1 = %x - 1
param %sub1
%rest = call @fact 1
%result = %rest * %x
return %result
@basecase: return 1
@main: %n = 3
param %n
%res = call @fact 1
return %res
```


I haven’t committed to this yet, but it seems reasonable. A bit more annoying for human authors, but I’m the only human author, and only for writing tests, so maybe I should stop caring so much about minor usability issues like that?

## Adding Stack Allocation Instructions
In C, it’s possible to create structs on the stack. However, Plachutta basically only offers heap allocation of memory (beyond local variables of a single “word”), via `memrequest`. Ironically, this is the exact opposite of what is offered by C, where functions like `alloc` are generally provided not by the language itself but by a library. I’m considering renaming `memrequest` to something like `halloc` (for heap allocation) and adding a new function, `salloc` (for stack allocation). This would be basically no work, since the only difference is which `Memory` instance it delegates to - global or local to the current `StackFrame`.
