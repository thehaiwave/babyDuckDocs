---
layout: default
title: Parser
nav_order: 2
---

# Parser

The parser itself is generated by ANTLR but the grammar is not. The DFA diagrams that were initially
provided to generate the grammar is shown here:

![alt-text](https://github-production-user-asset-6210df.s3.amazonaws.com/46984294/278871847-c1bce5d0-7be5-463e-96a3-4c194ee450d3.png)

The equivalent grammars for each of these tokens are shown as follows:

## Lexer tokens

```
Program         : 'program';
Main            : 'main';
End             : 'end';
Var             : 'var';
Void            : 'void';
Print           : 'print';
While           : 'while';
Do              : 'do';
If              : 'if';
Else            : 'else';
Int             : 'int';
Float           : 'float';
Identifier      : [a-zA-Z][a-zA-Z0-9]*;

Plus            : '+';
Minus           : '-';
Star            : '*';
Div             : '/';
Assign          : '=';
NotEqual        : '!=';
Greater         : '>';
Less            : '<';

LeftParen       : '(';
RightParen      : ')';
LeftBracket     : '[';
RightBracket    : ']';
LeftBrace       : '{';
RightBrace      : '}';
Comma           : ',';
Colon           : ':';
Semi            : ';';

CteString: '"' (~["] | '\\"')* '"';
CteInt: [0-9]+;
CteFloat: [0-9]+ '.' [0-9]+;

Whitespace: [ \t]+ -> skip;
Newline: ('\r' '\n'? | '\n') -> skip;
```

## Parser rules

```
programa: Program Identifier Semi vars? funcs* Main body End;

vars: Var (commaSeparatedId Colon type Semi)+;

commaSeparatedId:
    Identifier
    | (Identifier Comma)+ Identifier;


funcs: Void Identifier LeftParen idTypeSequence? RightParen LeftBracket vars? body RightBracket Semi;

idTypeSequence:
    Identifier Colon type
    | (Identifier Colon type Comma)+ Identifier Colon type;

body: LeftBrace statement* RightBrace;

type:
    Int
    | Float;

statement:
    assign
    | condition
    | cycle
    | f_call
    | print;

assign: Identifier Assign expression Semi;

condition: If LeftParen expression RightParen body (Else body)? Semi;

cycle: While body Do LeftParen expression RightParen Semi;

f_call: Identifier LeftParen commaSeparatedExpression? RightParen Semi;

commaSeparatedExpression:
    expression
    | expression Comma commaSeparatedExpression ;

print: Print LeftParen printSequence RightParen Semi;

printSequence: (expression | CteString) (Comma printSequence)?;

expression: exp ((Greater | Less | NotEqual) exp)?;

exp:
    termino
    | termino (Plus | Minus) exp;

termino:
    factor
    | factor (Star | Div) termino;

factor:
    LeftParen expression RightParen
    | factorSequence;

factorSequence: (Plus | Minus)? (Identifier | cte);

cte:
    CteInt
    | CteFloat;
```

There's not much to talk about here. One thing that might me of interest is the use of additional parser rules.
For instance, the use of `idTypeSequence` to remove some of the logic for the multiple variable declarations
away from the main `funcs` rule. This was done both to make creating the rules easier, but also because
of the way that ANTLR internally handles the resulting tree. The way it works is that if there are multiple
tokens of the same name in a single rule, they get put into an array. So for instance, consider the following
rule:

```
vars: Var ( Identifier | (Identifier Comma)+ Identifier; Colon type Semi)+;
```

ANTLR expose the different matched strings through listener functions in a class they provided when the parser
is generated. In this case, we would access `vars` through `exitVars` or `enterVars` depending on what we want
to do but that's the general idea.

The important part is this returned object contains a `getText()` function that returns the match for that specific
token, including all of the tokens it's made up of. So for instance, if our code has the following:

```
var
    funcBaz, funcQux, funcQuux: int;
    funcPlease: float;
    funcWork: float;
```

The object that ANTLR would give us for `vars` would look like this (after it has removed the spaces):

```
varfuncBaz,funcQux,funcQuux:int;funcPlease:float;funcWork:float;
```

We cannot do much with this. If we wanted to get the declared variable names and their types we would have
to do some manual matching and split the string somehow. Luckily ANTLR exposes something else to use, which
is functions that return the specific tokens inside this `vars` one. Our rule says we have a `Var` non-terminal,
and so ANTLR allows us to do:

```
enterVars(ctx){
    const varLiteral = ctx.Var().getText();
}
```

Basically, every token that makes up this `vars` token has their own object that provides data about it, like children
and, more importantly, the text literal. I have explained all of this to get to the important point which is that if we do
something like this:

```
const varId = ctx.Identifier().getText();
```

We would be doing something incorrect. See, the string that was matched contains multiple `Identifier` tokens. So
we need search them by index to find specific ones, like this:

```
const varId = ctx.Identifier()[0].getText();
console.log(varId)
// funcBaz
```

This presents a big problem. While we can indeed get the names of each of the variables, how should we get the types?
It should be easy to see now that they would be stored inside a context object like this:

```
const types = ctx.type()[0].getText();
```

See, both the variable names and types are stored inside arrays, but these arrays are of different sizes and we have
no way of knowing which names belong to which type declaration without doing manual matching by looking for
a colon character in the original string, splitting it and then matching the variable names that came before it
to the ones in our ANTLR object.

So instead, what we do is the following:

```
vars: Var (commaSeparatedId Colon type Semi)+;

commaSeparatedId:
    Identifier
    | (Identifier Comma)+ Identifier;
```

Since ANTLR doesn't do inline replacements of the tokens, even though this is matching the exact same strings as the rule
we had before, we can now access the variable names in such a way that we get one array for each of the types, instead
of a single array with all of the variable names.

```
// BEFORE
Identifier: [funcBaz, funcQux, funcQuux, funcPlease, funcWork]
Type: [int, float, float]


// NOW
Identifier: [[funcBaz, funcQux, funcQuux], [funcPlease], [funcWork]]
Type: [int, float, float]

```

This makes it trivial to match names to their types. Why does this happen? Because when ANTLR sees a token that
can be repeated multiple times, it automatically wraps the results in an array. Our first and second rule
both do this, but the difference is since now it is using a "helper" rule (`commaSeparatedId`), ANTLR will
put each "call" to it into its own array, resulting in the 3 different arrays for the 3 different name declarations.