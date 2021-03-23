---
layout: post
title: Adding Memory Operations to Plachutta
---
# Adding Memory Operations to Plachutta

In a previous post, I talked about [Plachutta], my interpreter for Three-Address Code. In that post, I mentioned that memory operations were one the bigger missing features (alongside functions). As I sat down to implement the new memory operations, I realized I didn’t understand the problem well enough to actually begin coding. This post is a cleaned up version of my planning and thinking about how to add memory operations.

## Is Is Necessary?
Before going any further, I want to confirm that I actually need this feature. In this case, I think it’s pretty apparent that I do. I want to support languages that have arrays and structs, and I think this is required for that. Otherwise, there’s no way to access anything other than the first element in the array/struct. No way around it - I need this.

## New Operators

### Indexed Access (`a[b]`)
Can appear either on the LHS or RHS of an assignment.

On the RHS, the value of `x[y]` is the value stored at the location in memory that is stored in (x +y). That is, if x contains the value 20, the value of `x[4]` is the value stored in memory location 24. `y` can be either an integer or a variable (although if it is a variable, it must point to an integer).

When used on the LHS, the indexed operator allows setting the location in memory pointed to by the value stored in the indexed variable, plus an offset. If we again assume that x contains the value 20, `x[4] = 5`sets the value in memory location 24 to be 5.

### Requesting Memory (`memrequest`)
Unary operator that asks the memory manager for a specific amount of contiguous memory. Its return value is the index of the start of that chunk of memory. Example usage:

```
x = memrequest 4
x[0] = 2
```

### What about `*` and `&`?
To keep things simple, I’m avoiding implementing `*` and `&`. It’s easy to implement dereferencing in terms of indexed operations - `*x` is equivalent to `x[0]`, both on the LHS and RHS of an assignment. Moreover, `memrequest` ought to avoid the need for the address-of operator `&`. They might be needed later on, but there’s no real purpose for them for now.

## Implementation Details
### Memory Representation
I will represent memory with a class Memory. The most important field in that class will be a Python List. The interface will look like:

```
ref(address, offset=0)
set(value, address, offset=0)
request(size, initial=None)
```

When variables are declared, they will be allocated a slot in memory. The mapping from a variable to its memory location will be tracked by a symbol table, which will be created by the analysis pass. The current `env` will be removed from the interpreter - instead, the value of a variable will be determined by looking up the memory location in the symbol table, then accessing that location via the `ref` function. The offset keywords will make implementing indexed operations trivial.

### `memrequest`
This should be trivially implemented by calling the `request` method of the `Memory` class. Internally, that will probably mean extending the list if necessary, then updating a counter that tracks the next free index. (To some extent, it depends on if I create a large list upfront, then extend it as necessary, or only extend the list as required.)

## Conclusion
This actually seems a lot more straightforward than I thought it might be! I think the first step is to convert the existing interpreter to use the new Memory class. Once that has been tested, I’ll implement the new operators.
