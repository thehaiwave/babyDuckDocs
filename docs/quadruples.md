---
layout: default
title: Quadruples
nav_order: 5
---

# Quadruples

This quadruple class is used to provide an interface to handle the produced quadruples
for the given program. Let's start by discussing the class state:

## Class State

```js
class QuadrupleGenerator {
  constructor() {
    this.globalQuadruples = [];
    this.jumpStack = new Stack();
    this.currentTemp = 0;
    this.loopStartIndex = 0;
  }
}
```

### `globalQuadruples`

An matrix holding quadruples. It is an array of arrays, with each array representing a quadruple of the format:

[operator, operand1, operand2,...]

The exact length of the quadruple depends upon que quantity of operands needes for the given operator. Some, like
the `print` only require one, while others like expressions require two operands plus a temp variable.

### `jumpStack`

A stack that keeps track of positions we need to jump back to. It is to set GOTOs and to at a later evaluation of
another quadruple go back and set the value.

### `currentTemp`

Keeps track of the current count for the temp variables used throughout the program, so that they consistenly follow
the pattern of `t0, t1, t2, t3...`.

### `loopStartIndex`

Used for the condition token. We use this state to save the variable that we need to GOTOV to from the DO...WHILE
conditional statement.

## Methods

### `genTempId`

Function used to generate the temp variables for the quadruples. It just appends a "t" character to whatever is in the
`currentTemp` variable.

### `addToQuadruple`

Function that takes in a raw quadruple and flattens it to make the elements easily accesible before pushing it to
`globalQuadruples`.

### `genAssignQuadruple`

It takes in an array of normalized characters and a memory object. Like the name says, it just evaluates an expression
and generates the quadruples for it. It also returns the last temp variable it used so that we can use it to evaluate
conditionals and prints.

### `genExpressionQuadruple`

Does the same as `genAssignQuadruple` but without taking care of the `=` symbol. In general, both this and `genAssignQuadruple`
work fundamentally the same. They iterate through an array of objects that look like this:

{name: 'someName', type: 'some-type'}

And using an operand and operator stack, they decide what to do when seeing an object like the one above. For instance, if it
sees an operator whose type is `/`, it will add it to the operator stack, if it sees an object whose type is `Identifier` or
`CteInt` it will add them to the operand stack.

Now, the algorithm for this gives division and multiplication the second highest priority. Basically, every time we come accross either
of these 2 symbols we immediately evaluate, but we cannot do this when seeting the operator itself because it is a binary operator,
we need 2 symbols. So, instead every time we come accross an operand we ask if the top of the operator stack contains a multiplication
or a divison. If it does, we pop it, along with 2 operands from the operand stack and add a quadruple immediately.

The idea is to leave only subtractions and additions that we can evaluate from right to left by recursively popping the operand
and operator stacks. This is not always possible because we also have to support parenthesis. These are the highest priority operator,
and every time we come accross an opening parenthesis we start keeping track of when it closes to we can immediately start creating
quadruples for everything that's inside of it before continuing with the expression.

By following the above steps we can correcty evaluate expression and resolve them to quadruples.

### `getQuadruples`

Returns the global quadruples.

### `insertQuadruple`

Inserts a quadruple

### `setGoto`

Sets a GOTO in the quadruple array and initializes the GOTO line number as null so it can be set later.

### `bringGoto`

Gets the last GOTO in the `jumpStack` and assigns it the current quadruple as the GOTO value. So if this
function is called on the 5th quadruple, the GOTO will be to the 5th quadruple. An offset can be optionally
provided to move this value around relative to the quadruple it was called on.

### `cycleJump`

Saves the quadruple line where a cycle begins. This will later be referenced when `genConditionalJump` so we know
where to send the GOTOV.

### `genConditionalJump`

Creates a GOTOV in the current quadruple. Takes in the variable to use for the condition and gets the `cycleJump` value
to immediately know where it needs to go.
