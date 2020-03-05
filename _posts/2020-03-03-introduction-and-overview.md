---
layout: post
title: "Day 0: Introduction and Project Overview (a.k.a. Hello World)"
date: 2020-03-03
categories:
---
## My Background
Every time I contemplate working on personal software projects, I almost always come back to writing a compiler. To date, the piece of software I'm most proud of is the Scheme -> x86-64 compiler I wrote as an undergraduate, teaching myself from the assignments for the P423 Compilers course, with help from the professor and grad student I was hoping to help with research. I also sat in on the weekly Programming Languages group presentation, and helped teach the undergraduate programming languages course after taking it.

After graduating, however, I kind of gave up on those parts of programming. I was working as a new engineer at large tech company, which felt like a whole new world. In college, I had been writing compilers in Lisp, and now I was writing huge distributed systems in Java. I struggled a lot in that role, and found that it was almost impossible to motivate myself to work on personal coding projects. Instead, I mostly wanted to just get through the day. I eventually left, and found my love of programming again, mostly by reading about and attempting to implement programming langauges. Along the way, I picked up Javascript and hugely improved my Python, started a couple of my own companies, watched those companies fail, and eventually got a new job at a startup writing distributed systems once again (this time in Python instead of Java, which was a big improvement). Once again, I found it hard to stay focused on keeping up with new tech developments. The team and company were great, but I found myself looking for ways out of engineering and into something else, like product or people management.

Last week, I left that role. The day after I left, I found myself sitting at a coffee shop, and for the first time in months, maybe years, I wanted to work on a personal programming project. I settled on a small interpreter written in Python, and it felt like when I first learned to program. My PL fundamentals were a little rusty, but my improved abilities as an engineer more than compensated.

While idly searching for new roles last week, I stumbled upon something that it had never occurred to me to look for: a position writing compilers professionally. I had never thought that such a position was an actual possibility. I mean, intellectually, I knew jobs like that must exist, but I had never stopped to ask myself what things I would need to do to actually get such a role.

## Purpose of this Blog
This blog, then, is an attempt to figure out what things I should do, then chart my journey as I do those things. I have 5 related reasons to blog about it:
1. To keep myself accountable
2. To force myself to do some upfront planning on what I'm trying to do, and to encourage regular reflection on what I have learned and accomplished recently
3. To track my progress so I can look back later and remind myself how far I've come.
4. To force myself to understand things well enough that I can explain it in public to strangers
5. To be the resource I wish I'd had after graduating college

## Plan
I'm still in the planning phases, but as of this second my plan is to spend the next several months learning everything I can about compilers. Even when I was an undergrad doing research, I had too many other obligations to really dive as deep as I might have. Right now, I have a unique opportunity to focus without distractions for multiple months on a single topic. My ideal deliverable would be a self-hosting C compiler, thoroughly documented and described in detail on this blog.

I also want to work my way through the [Dragon Book][dragon-book-wiki]. Scheme has a lot of advantages as a langauge for writing compilers, but one weakness is that lexing and parsing are basically trivial - a first year CS student can pretty easily write a Scheme reader, but they would never have to because the built-in `read` function can do all the heavy lifting. As a result, lexing and parsing are huge gaps in my understanding. Additionally, most of my knowledge of the frontend for a compiler is focused on supporting a functional programming language. I don't yet know how C's different paradigm will affect things, but it will surely be different. I've also never written a runtime system, so I'll need to fill that fairly large gap in my knowledge as well. While I know a fair bit of type theory, I've never actually implemented a type checker of any complexity. All those things will be totally new. Even for the things I do know (fundamentals like register allocation or instruction selection), it's been a long time since I've thought about them in detail. And finally, my knowledge of C and x86 assembly are workable but rudimentary, so I expect I'll have to bulk up my knowledge there.

Here's an optimistic schedule to give a rough sense of things:
### Already Done
* Gathering some resources (books on C, x86 assembly, and compiler design)
* Setting up a pleasant development environment for writing C on a MacBook (using Docker, which works excellently)

### This Week
* Define the initial subset of C I'm going to target
* Lexer theory (Chapter 3 of Dragon book)
* Write a Flex lexer for a small subset of C
* Career development outreach (e.g. reaching out to people who have the type of jobs I want and asking them for advice)

### This Month
* Parsers (Chapter 4 of Dragon book)
* Write a parser in bison for my subset of C
* Finish parsers
* Generate Three Address Code for my subset of C
* Finish backend chapters of Dragon book (no optimiations, no type checking, no runtime system)
* Write passes for exposing basic blocks, allocating registers, instruction selection, assembly generation, etc.

### Future Months
* Finish Dragon book
* Type checker
* Runtime system
* Expanding the language to support more of C
* Integrating with existing C libraries
* Optimizations! (This is what I'm most excited about, kind of bummed it's going to take so long to get to. I might work some of this in earlier just for fun.)
* A ton of other stuff I'm sure I'm missing...

## Conclusion
That's it for now! I'm currently working through Chapter 2 of the Dragon book, which implements a very simple infix-to-postfix compiler. It's a *long* chapter, and introduces a bunch of important concepts. Currently, I'm working on the parser for that langauge. For those who want to follow along, I'm publishing my progress to a [Github repo][dragon-book-c-repo].

[dragon-book-wiki]: https://en.wikipedia.org/wiki/Compilers:_Principles,_Techniques,_and_Tools
[dragon-book-c-repo]: https://github.com/adhearn/dragon-book-c
