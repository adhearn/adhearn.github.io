---
layout: post
title: "Day 1: Interesting Things I Learned"
date: 2020-03-04
categories:
published: false
mathjax: true
---

## Transforming Left-Recursive Grammars
First off, I didn't know what a left-recursive grammar was, so let's start with that. A left-recursive grammar is a grammar where a nonterminal has a production where the first term in the production is recursive. For example, the following grammar, for a simple language representing addition and subtraction of single-digit non-negative integers, is left-recursive:

$$
\begin{align*}
    & expr: expr + digit \\
    & \qquad \vert \; expr - digit \\
    & \qquad \vert \; digit \\
    & digit: 0 \; \vert \; 1 \; \vert \; ... \; \vert \; 9 \\
\end{align*}
$$

However, this type of grammar isn't parseable by a naive predictive parser. In a predictive parser, the parser starts at the first nonterminal in the grammar. In the example above, that would be `expr`. It also sets up a lookahead, which will point at the first terminal returned by the lexer. Our given language is so simple that all of our terminals are single characters, so we can just have it point at the first character of the input string. Whenever our parser processes a terminal (either a digit, '+', or '-'), we will advance the lookahead one character.

So, consider the input string "9+4". Our lookahead will be '9'. We'll enter at the `expr` nonterminal and try to parse that. Our parser will try to apply the first production, and will recursively call expr *without* advancing the lookahead, since we haven't yet encountered a terminal. Once again, the parser will try to apply the first production without advancing the lookahead, and once again it will recur. With our simple parser, we simply cannot correctly parse the given grammar.

All is not lost! We can transform the grammar to be parseable with our simple predictive parser. The rule to follow is that if you have a grammar with a production like $$ A \rightarrow Aa \; \vert \; b $$, you can introduce a new nonterminal to remove the left recursion. The new grammar will look like:

$$
\begin{align*}
    & A \rightarrow bR \\
    & R \rightarrow aR \;  \vert \; \epsilon  \\
\end{align*}
$$

The grammar we started with can be transformed to:

$$
\begin{align*}
    & expr: digit \; rest \\
    & rest: + \; digit \; rest \\
    & \qquad \vert \; - \; digit \; rest \\
    & \qquad \vert \; \epsilon \\
    & digit: 0 \; \vert \; 1 \; \vert \; ... \; \vert \; 9 \\
\end{align*}
$$

Neat!


## Parse Trees (Concrete Syntax Trees) and Abstract Syntax Tress Are Different
When I first saw parse trees, I immediately thought that they were the same as syntax trees. They are not! In a technical sense, that's because the non-leaf nodes of a parse tree are nonterminals, whereas in an AST the non-leaf nodes represent programming constructs. However, even after reading that definition, I thought that differentiating them was just being pedantic. Not so! Consider the non-left-recursive grammar we specified above and an input string `9+4`. The AST for that input is simple: <figure out how to show ASTs in markdown>. The parse tree is now more comx`plex: <show the parse tree, including the epsilon node>. Clearly, not the same tree!
