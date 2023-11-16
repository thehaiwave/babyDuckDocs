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

## SemanticChecker

This is a class containing a single method.

### `transformAntlrToArray`

This method takes in an array containing the segment of the token that was matched in the listeners. It also
takes in the start and end indices for this segment, such that the function can slice the segment from the global
array of tokens and convert them to a usable notation to work with based on the `TokenTypes` object.
