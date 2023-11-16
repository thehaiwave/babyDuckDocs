---
layout: default
title: Interpreter
nav_order: 7
---

# Interpreter

The interpreter takes is a class that runs the quadruples and evaluates them.

## Class State

```js
constructor(quadruples, memory) {
    this.quadruples = quadruples;
    this.memory = memory;
    this.currentQuad = 0;
 }
```

This class is very simple. It takes in a matrix of quadruples and a memory instance. It will use the former
to iterate through them and evaluate them and the latter to perform the actual evaluations, since the memory class
contains the methods needed to perform multiplications, assignments, etc.

`currentQuad` is simply an instruction pointer that points to the quadruple being currently evaluated.

## Methods

### `run`

Main function. It is invoked directly from whatever file instantiates this class. It is a loop that will keep calling
`executeQuadruples` passing the quadruple being currently pointed to by the instruction pointer. It will exit if the instruction
pointer was set to `null` by one of the quadruple evaluations.

### `executeQuadruples`

Takes in a quadruple. It then grabs the first element which is always an operator and goes through a switch statement. For lineal
statements it simply calls the corresponding function in the `Memory` class with the necessary arguments and increases the
instruction pointer by one. When dealing with non-linearl statements like IF statements and cycles, it will manually
set the instruction pointer to a specific quadruple (to support GOTOs) depending on conditions specific to that construct.
For instance, GOTOFs and GOTOVs need to call certain methods in the `Memory` class to know what to do.
