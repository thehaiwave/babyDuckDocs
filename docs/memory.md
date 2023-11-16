---
layout: default
title: Memory
nav_order: 6
---

# Memory

This class saves and manages the memory of every single variable in the quadruples.

## Class State

```js
class Memory {
  constructor() {
    this.memory = {};
    this.intCounter = 0;
    this.intUpper = 999;
    this.floatCounter = 1000;
    this.floatUpper = 1999;
    this.temporalBoolCounter = 2000;
    this.temporalBoolUpper = 2999;
    this.temporalIntCounter = 3000;
    this.temporalIntUpper = 3999;
    this.temporalFloatCounter = 4000;
    this.temporalFloatUpper = 4999;
    this.funcCounter = 5000;
    this.funcUpper = 5999;
  }
}
```

As you can see, we don't keep multiple objects for each of the types, so basically we store all floats, ints, temps, funcs, etc.
in the `memory` object. In order to simulate memory addressed we establish a couple of variables for each of these types
to keep track of the ranges of valid memory addresses, meaning our memory is "segmented" (not really, but it behaves as if it was).

This decision was make to make it easier to index variables. If we kept multiple objects we'd have to carry around an object
with the symbol and type in the quadruples. This way, we only have to deal with a string with the symbol name and index it once
in the memory object.

## Methods

### `resolveVar`

It resolves a var a returns its type if it exists, otherwise it returns null. This function is used for semantic checking purposes,
we are only interested in verifying that the types of the symbol are allowed in any given binary operation so we are not
interested in the symbol names.

### `addTempVariableToRegister`

Adds a symbol to the memory. It differs from `addVariableToRegister` in that it can set boolean value types, while `addVariableToRegister`
can only handle integers and floats.

### `getTempVarMemoryAddress`

Checks the memory object to see if there are any memory addressed available for the given symbol's type. Deals with booleans.

### `insertTempVariable`

Allocates a memory address for the temp variable, creates an entry for the memory object and inserts it in the memory.

### `checkMemoryTypes`

Function used to check if a binary operation can be performed with two given operands based on their type. The function
first tries to resolve both symbols before checking the semantic cube for the return type and verifying it doesn't
return an error. If everything is good, it returns the return type of the binary operation.

### `getVarMemoryAddress`

Checks the memory object to see if there are any memory addressed available for the given symbol's type. Does not deal with booleans.

### `addVariableToRegister`

Adds a symbol to the memory.

### `validateSymbol`

Checks that the given symbol is not already present in the memory. Throws an error if it is.

### `printMemory`

Prints the memory object.

### `resolve`

Resolves the memory object for a given symbol name. Returns entire object not just type.

### `assignMemory`

Used to set memory when dealing with assignments when evaluating the quadruples. Will check to see if both symbols exist and
proceed with the assignment if they do.

### `addSum`

Takes in two symbols and a temp variable. It adds the symbols after checking both exist and assigns the result to the temp variable
in memory.

### `comparison`

Takes in two symbols and a temp variable. It compares the symbols after checking both exist and assigns the result to the temp variable
in memory.

### `gotocond`

Takes in a symbol. It returns that symbol's boolean value after checking it exists.

### `print`

Takes in either a string literal or a variable object, in which case in prints its value.

### `mul`

Takes in two symbols and a temp variable. It multiplies the symbols after checking both exist and assigns the result to the temp variable
in memory.

### `div`

Takes in two symbols and a temp variable. It divides the symbols after checking both exist and assigns the result to the temp variable
in memory.

### `sub`

Takes in two symbols and a temp variable. It subtracts the symbols after checking both exist and assigns the result to the temp variable
in memory.

### `gte`

Takes in two symbols and a temp variable. It compares the symbols after checking both exist and assigns the result to the temp variable
in memory.

### `getFuncMemoryAddress`

Checks if there is still a memory address available to assign to a function. Returns one if so, otherwise throws error.

### `insertFunc`

Inserts function into memory object.

### `checkArgType`

Runs verifications on the arguments when making a function call, like making sure the argument types match the ones on the
function's declaration, checking for the number of arguments, checking if the function being called exists, etc.
