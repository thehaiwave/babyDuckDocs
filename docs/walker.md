---
layout: default
title: Tree Walker
nav_order: 4
---

# Tree Walker

## The Class

When creating the ANTLR parser, one of the classes it automatically generates is a listener clas.
In my project, this class can be found under `./parser/babyduckListener.js`. This class by itself
only provides a couple of listener methods of the `enterNode` and `exitNode` for every single
one of our grammar tokens. That is to say, if our grammar consists of a `main` and `body` token,
this class will give us back:

-`enterBody`

-`exitBody`

-`enterMain`

-`exitMain`

Essentially, we can pass callbacks for when the listener class starts and finishes matching any
given token. This proves useful when handling context, but more on that later.

Now, this class is useless by itself. What ANTLR allows us to do is extend this class and implement
the content of these callbacks ourselves. For this, I created the class `Porfavor` which
extends the `babyduckListener` class. There's a caveat to this though. Although the class
gives us a `ctx` object passed to every listener function containing data about the matched token,
its parent, children, etc., we cannot actually orient ourselves in the program with just this
information. The `ctx` object gives us a very limited amount of data, and it is meant to be used
to perform relatively simple operations upon the grammar. Crucially, it does NOT give us the types
for the tokens. It does not tell us if a token is a `CteString` or an `Identifier` directly, for instance.
We have to convert a type ID to a string to make this information usable.

The solution to this is to work with the array of tokens of the entire program itself, which looks like
this:

```js
const tokens = new antlr4.CommonTokenStream(lexer);
const tokenObjects = tokens.tokens;
```

Let's say we get a program like this:

```
main Programa;
    main{};
end
```

Then, `tokenObjects` would look like this:

```
[
  Dt {
    source: [ [babyduckLexer], [ke] ],
    type: 1,
    channel: 0,
    start: 0,
    stop: 6,
    tokenIndex: 0,
    line: 1,
    column: 0,
    _text: null
  },
  Dt {
    source: [ [babyduckLexer], [ke] ],
    type: 13,
    channel: 0,
    start: 8,
    stop: 26,
    tokenIndex: 1,
    line: 1,
    column: 8,
    _text: null
  },
  Dt {
    source: [ [babyduckLexer], [ke] ],
    type: 30,
    channel: 0,
    start: 27,
    stop: 27,
    tokenIndex: 2,
    line: 1,
    column: 27,
    _text: null
  },
  Dt {
    source: [ [babyduckLexer], [ke] ],
    type: 2,
    channel: 0,
    start: 34,
    stop: 37,
    tokenIndex: 3,
    line: 3,
    column: 4,
    _text: null
  },
  Dt {
    source: [ [babyduckLexer], [ke] ],
    type: 26,
    channel: 0,
    start: 39,
    stop: 39,
    tokenIndex: 4,
    line: 3,
    column: 9,
    _text: null
  },
  Dt {
    source: [ [babyduckLexer], [ke] ],
    type: 27,
    channel: 0,
    start: 45,
    stop: 45,
    tokenIndex: 5,
    line: 4,
    column: 4,
    _text: null
  },
  Dt {
    source: [ [babyduckLexer], [ke] ],
    type: 3,
    channel: 0,
    start: 47,
    stop: 49,
    tokenIndex: 6,
    line: 5,
    column: 0,
    _text: null
  },
  Dt {
    source: [ [babyduckLexer], [ke] ],
    type: -1,
    channel: 0,
    start: 51,
    stop: 50,
    tokenIndex: 7,
    line: 6,
    column: 0,
    _text: null
  }
]
```

Firstly, this is a much more useful array than the one `ctx` provides us for a token's children
because it is flat, we don't gave to recursively dig through children properties and test
each one until we find the one we need. Secondly, notice the `type` property for each of this.
Each of these integers is mapped to an actual value in our grammar. For BabyDuck, this is the
map, with the -1 internally signifying an EOF:

```js
const TokenTypes = {
  Program: 1,
  Main: 2,
  End: 3,
  Var: 4,
  Void: 5,
  Print: 6,
  While: 7,
  Do: 8,
  If: 9,
  Else: 10,
  Int: 11,
  Float: 12,
  Identifier: 13,
  Plus: 14,
  Minus: 15,
  Star: 16,
  Div: 17,
  Assign: 18,
  NotEqual: 19,
  Greater: 20,
  Less: 21,
  LeftParen: 22,
  RightParen: 23,
  LeftBracket: 24,
  RightBracket: 25,
  LeftBrace: 26,
  RightBrace: 27,
  Comma: 28,
  Colon: 29,
  Semi: 30,
  CteString: 31,
  CteInt: 32,
  CteFloat: 33,
  Whitespace: 34,
  Newline: 35,
  program: 1,
  main: 2,
  end: 3,
  var: 4,
  void: 5,
  print: 6,
  while: 7,
  do: 8,
  if: 9,
  else: 10,
  int: 11,
  float: 12,
  "+": 14,
  "-": 15,
  "*": 16,
  "/": 17,
  "=": 18,
  "!=": 19,
  ">": 20,
  "<": 21,
  "(": 22,
  ")": 23,
  "[": 24,
  "]": 25,
  "{": 26,
  "}": 27,
  ",": 28,
  ":": 29,
  ";": 30,
};
```

It would be a much cleaner way of working if we could turn the `tokenObjects` into something more standarized and
tailored to the operations we will be performing on the tokens but for now, we will only worry about passing
this `tokenObjects` to our `Porfavor` class. After all this preamble, we arrive at this:

```js
const listener = new Porfavor(tokenObjects, memory);
antlr4.tree.ParseTreeWalker.DEFAULT.walk(listener, tree);
```

## The Walker

`Porfavor` is the name of our tree walker, though it is perhaps best described as a listener class.
Right off the bat, we need to define some state for this class which we will use to create the
quadruples. In our case, we have this:

```js
class Porfavor extends babyduckListener {
  constructor(tokenObjects, memory) {
    super();
    this.tokenObjects = tokenObjects;
    this.contextStack = new Stack();
    this.QuadrupleGenerator = new QuadrupleGenerator();
    this.SemanticChecker = new SemanticChecker();
    this.Memory = memory;
    this.currentContext = null;
    this.seenMain = false;
    this.startingElse = false;
  }
}
```

### `tokenObjects`

An array of objects, each representing a token in our input program.

### `contextStack`

Stack that keeps track of the current context.

### `QuadrupleGenerator`

Instance of the `QuadrupleGenerator` class which holds functions to handle the quadruples, from creating them,
fetching them, printing them, inserting them, calling them, etc.

### `SemanticChecker`

Instance of the `SemanticChecker` class which holds the semantic cube and some utilities to work with the
semantic checking of the tokens.

### `Memory`

Instance of the `Memory` class. This class holds our virtual memory for integers, floats, functions and also
holds utility functions for inserting them and checking for memory spaces when doing so.

### `currentContext`

Variable that keeps track of the latest context.

### `seenMain`

Flag to help determine if we are dealing with the main body function of the program.

### `startingElse`

Flag to help determine if we are dealing with a condition that has an `else` clause.

## The Functionality

The idea for this class is very simple. Every time we enter or exit a block, we perform some
sort of operation, be it setting context, generating quadruples, setting GOTOs, etc. What
exactly is done depends heavily on the nature of the block, but for now let's worry about
the general flow of a program.

So, every BabyDuck program must start with a `program`, some identifier, vars, funcs, etc.
To not make this too long here's the grammar:

```g4
programa: Program Identifier Semi vars* funcs* Main body End;
```

Every single token we define like this in our G4 ANTLR file has enter and exit functions. So,
we start out our program by entering this;

```js
  enterPrograma(ctx) {
    this.contextStack.push("PROGRAMA");
    this.QuadrupleGenerator.setGoto([["GOTO", null, null]]);
  }
```

We start out by setting the global context `PROGRAMA` and setting a GOTO. This GOTO is necessary because
we read and parse our program line by line, but the main function won't be at the start so we need to make
our very first quadruple a GOTO to jump to that main as soon as we find it. Now, the next token will get called
since non-terminals do not get their listener functions. In this case, that next token will be `vars`. Let's
check that out:

```js
  enterVars(ctx) {
    const varsContext = ctx.parentCtx.Identifier().getText();
    const typeQuant = ctx.commaSeparatedId().length;

    for (let i = 0; i < typeQuant; i++) {
      const varsType = ctx.type()[i].getText();
      const varNumbers = ctx.commaSeparatedId()[i].Identifier().length;
      if (varNumbers > 1) {
        for (let j = 0; j < varNumbers; j++) {
          const varName = ctx.commaSeparatedId()[i].Identifier()[j].getText();
          this.Memory.allocateVariable(varName, varsType, varsContext);
        }
      } else {
        const varName = ctx.commaSeparatedId()[i].Identifier()[0].getText();
        this.Memory.allocateVariable(varName, varsType, varsContext);
      }
    }
  }
```

So a couple of things are happening. First, we are getting the context of the program through the
`ctx` object. We KNOW that the parent context will be the `programa` token we defined earlier, so
it is guaranteed that this will have an `Identifier`, which we can call with `Identifier().getText()`.
Next, we need to know how MANY vars are being declared. I have a custom token called
`commaSeparatedId` that looks like this:

```g4
commaSeparatedId:
    Identifier
    | (Identifier Comma)+ Identifier;
```

If a token can be inserted multiple times, ANTLR puts all of those token's instances in an array, so getting
the length of it is trivial, and we can deduce the number of types and vars for each type by doing
some clever looping. I won't get into detail, but we basically iterate for every one of the identifier lines
separated by comma ending in a type, so basically mathing this:

```g4
var
    // matching ONLY this, there can be multiple lines of commaSeparatedId
    n, first, second, next, count: int;

```

For each of those comma separated we then get the name and the type, and we allocate memory for it. Some checks
are ran for this, but I will elaborate on those in the Memory's documentation page.

And that's basically it. We run a similar process for every single one of the important tokens,
setting and removing context based on what we need at the time. The code can be consulted
for the rest of the listeners implemented. The class itself only deals with managing the context and the
tokens for that matched string, but the magic happens in the other classes.
