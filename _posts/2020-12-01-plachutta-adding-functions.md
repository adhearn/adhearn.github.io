---
layout: post
title: "Adding Functions to Plachutta"
---
Like with the memory operations (which are now complete!), this is intended to help me think through how to add functions to Plachutta. My best guess is that functions are going to be significantly more difficult than memory, so expect this to be longer and more complicated as well!

## Is It Necessary?
Yep, no getting around this one.

## New Operators
### `param <address>` (solo)
Pushes a value onto the parameter stack. (Calling convention is call-by-value.)

### `<variable> = param <address>` (RHS of assignment)
Pops a variable off the parameter stack and assigns it to a variable.

### `call <fn> <num_params>`
Calls a function with the specified number of parameters.

### `return <address>`
Returns a value to a caller.

## Implementation Details

### Parameters
Our interpreter needs to track the parameters that have been pushed onto the stack. Easily represented with a Python list. When a parameter is added, we’ll push its value to the stack. When we process a call, we’ll pop the specified number of arguments from the parameters stack and copy them into the StackFrame parameter stack (see below). That ensures that  all the parameters are popped off the stack after a return even if the called function doesn’t do it.

### Stack Frames
When actually writing assembly, we generally push the return instruction address onto the stack so that we know where to jump when the function returns. Strictly speaking, we don’t actually need to do that, because Python already did it for us. However, I’m going to include it because it could be useful in debugging. The stack can also be used for storing function call parameters and local variables. We copy parameters from the parameter stack to the stack frame on function call, and however we end up handling local variables, we’ll probably use stack frames to do it.

### The Scope Problem
So far, I’ve been operating under the assumption that everything will work, but there’s a problem. Consider the following program, which is intended to calculate 3!:

```
fact: x = param
if x <= 1 jump basecase
sub1 = x - 1
param sub1
rest = call fact 1
result = rest * x
return result
basecase: return 1

main: n = 3
param n
res = call fact 1
return res
```

Assuming that the interpreter uses the `main` function as its entry point, what will the program return?

The first time through `fact`, x will be set to 3, since that’s the parameter that was passed in. We’ll calculate sub1, which will be 2, push that onto the parameter stack, then recursively call `fact`. This time, we’ll set x to 2, calculate sub1 to be 1, push that to the parameter stack, then call `fact` once more. x is 1 this time, so we jump to the `basecase` label and return to the caller. We set rest to the return value, which was 1, then calculate result, which is `rest * x`. However, x is now 1! When we updated it in the innermost call to `fact`, it was updating globally, even though when we wrote this program we really wanted it to just update a local version.

In a higher-level language, we would rely on function scoping rules to resolve this issue, but TAC is too simple to have true functions - it just has jumps to labels. In assembly, we could refer to local variables by their offset from the stack pointer. That way, when we enter a new function, the stack pointer has changed and we’re accessing a different spot in memory. However, that’s lower level than what I want for TAC (at least if I can help it - I haven’t thought through all the implications yet). I see two possible solutions.

#### Declare Globals as Global
One solution would be to declare certain variables as being global at the beginning of the program by adding a new global instruction. The symbol table would then track which variables were global and therefore know if it needed to look in the global environment or the local environment to retrieve or update the value of any given variable. The interpreter itself wouldn’t even have to care about the details - it would just ask for the value of the variable and the symbol table would handle the rest.

This is actually a pretty reasonable solution to support a language like C. There’s global scope and non-global scope, and you’re either in one or the other. Globally scoped variables are declared ahead of time, so it would be easy to translate into TAC. This was my initial solution, and still the one I’m leaning toward for the time being.

#### Closures All the Way Down
The more functional approach would be to pretend that global variables didn’t exist. Instead, a function that needed to access a global variable could be explicitly passed that variable as a parameter. If I expected to be targeting TAC with functional languages, I would be more inclined to go down this route. However, it seems like a poor fit as an IR for a functional language, so I don’t want to invest too much effort.

## Operators to Handle Scope

### `global <variable>`
Declares a variable to be global. Any time this variable is encountered, the global environment will be checked.

## Scope Implementation Details and Changes
1. `Variable` class will be updated to have a boolean field, `global`. If the variable is a global variable, this will be set to True, otherwise False.
2. The Analysis pass will be updated to recognize new `global` instruction. When it encounters a global, it will update the symbol table to set the `global` field to be True.
3. `initialize_memory` (the function responsible for setting up memory locations for all variables) will no longer create memory locations for all variables. Instead, it will just allocate space for globals.
4. Create StackFrame class, which will have fields for return instruction address, parameters, and a local Memory instance. There will be a private instance variable that also tracks name-to-memory mappings within the context of this specific StackFrame.
5. RHS variable value access will first check if the `global` flag of the Variable instance is set. If it is true, we’ll use the `memory_location` field on the Variable. Otherwise, we’ll look up the location via the StackFrame.
6. Similarly, LHS of assignments will look at the global field Variable object to see if they should set the value in global Memory or in the current StackFrame.
All of this can be done regardless of the other new operations.

## TAC Program Structure
Now that we have functions, we can enforce rules like “to be interpreted, a valid TAC program must have a function `main` that will be the entry point to the program. The return value of the program is the value returned from the `main` function. If there is no `main` function, the return value of the program will be None.” In fact, that’s the exact rule I want to enforce. We’ll add that rule as the final step after we add the required functionality.

## Overall Plan
1. Add new `global` instruction and update interpreter to handle new rules (i.e. all the stuff in “Scope Implementation Details and Changes”).
2. Add `param`, `call`, and `return` functionality.
3. Enforce that programs must have a `main` label, and update the interpreter to jump to the `main` label as first instruction to process.

Once all those things are done, I will consider this phase complete.

## Final Thoughts
A lot of the complications to handle scope are only really required because we’re trying to make TAC a standalone language. If it were just an intermediate phase of a compiler, we could rely on the symbol table already knowing which variables were global and which were local. I was a bit frustrated that no one that I could find bothered to write about how to represent these scoping rules, but that’s because most of the time you don’t need to worry about it!

A totally alternative approach would be to split TAC into two separate files - a symbol table and instructions. Instead of building the symbol table by parsing the body of the program, our language could build the symbol table just by reading the symbol table file. To make life a little easier, you could put them in a single file, perhaps having a symbol table “header” that would give directives about how to construct the symbol table, then the body of the program. If you moved all the `global` instructions to the top of the file, that’s actually pretty close to how we’re implementing it, we just make the programmer’s life easier by adding an analysis pass that identifies all the other variable names to construct the rest of the symbol table. Since I’m the only programmer (lots of tests!) I might change this in the future once compilers become the primary “programmers” of the language.
