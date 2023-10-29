---
layout: default
title: Syntactic Analysis
nav_order: 3
---

# Syntactic Analysis

## Cubo semantico

This is defined [here](https://github.com/thehaiwave/BabyDuck/blob/main/syntactic_analyzer/babyduckSemanticCube.js).
The chosen data structure for this is a JavaScript object. The reason for this is that this cube is static
and will never have elements added to it at runtime or buildtime, so I do not have to worry about making
it extremely efficient, it only needs to handle indexing fast which is what JS objects are good at since
they are pretty much hashmaps.

The only thing I feel the need to talk about is my choice of allowing
arithmetic with booleans. This is not specified anywhere in the project requirements, but while
analyzing the grammar I created the following test for it:

```
innerQuux = 30 / 2.5 + -450 > (250 != 100) * -600 - 900;
```

This is valid in Babyduck, and so I am working on the assumption that boolean values
can be cast as integers when performing arithmetic with it, like Python does. In fact, this
returns `True` in Python, and the only reason that is possible is because Python is casting the result
of `(250 != 100)` to an integer `0`. This is something that I can take care of, but it took some
time to accurately replicate the behavior of this in my syntactic cube. Some operations with
casted boolean values actually return floats, for instance.

## Directorio de funciones y tabla de variables

Can be found [here](https://github.com/thehaiwave/BabyDuck/blob/main/funcDir.js) and [here](https://github.com/thehaiwave/BabyDuck/blob/main/varTable.js)
respectively.

These have been defined as JS classes in order to keep track of the stored symbols and encapsulate some behavior when dealing with them.
For now, the only 3 actions that can be performed by either of them are to add a symbol, delete a symbol or return the stored symbols.
For both of them we check whether the symbol has been previously added and if so we throw an error. In the case of the functions
we actually also want to check if the arguments are repeated, so basically preent this from happening:

```
void foo(argFoo: int, argFoo: int)
```

I do not actually check for this inside of the `funcDir` class since I would have to pass some very large ANTLR objects as parameters
and it would get a bit messy. Instead, I check for it inside of my [`CustomListener`](https://github.com/thehaiwave/BabyDuck/blob/main/main.js#L9)
class, which deals with the actual token matches.

For both of these I chose to use a JavaScript object. Insertions to it run in constant time because it's basically a hashmap
so it is incredibly fast for fetching and adding elements to it. There is no set structure defined for it when the class
is initialized, I simply format the function or variable data when adding it and push it to the object to store it.
