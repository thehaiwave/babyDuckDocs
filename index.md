---
title: Baby Ducks Docs
layout: home
nav_order: 1
---

# Babyduck

Babyduck is an imperative procedural language. This particular website details and documents
the parts of a compiler written by Carlos Brito | A01283561.

[Get started now](#getting-started){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View it on GitHub](https://github.com/thehaiwave/BabyDuck){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## Getting started

This project was created using [ANTLR4](https://www.antlr.org/) and the JavaScript [target](https://github.com/antlr/antlr4/blob/master/doc/javascript-target.md)
for it. Like the documentation says, all that's needed to create a parser based on a grammar provided by you, you only need to install the tool:

```
pip install antlr4-tools
```

And then, ANTLR4 _only_ requires that you provide a grammar file (a file ending in `.g4`) with your grammar on it. In my case, my file is called `babyduck.g4`, so in the same directory where this file is I only need to run:

```
antlr4 -Dlanguage=JavaScript babyduck.g4 -o parser/
```

When using targets that aren't Python we need to specify the language, in this
case JavaScript and then optionally an output directory. This entire directory
can be deleted and regenerated when changes to the grammar file are made.

### My compiler

Clone the repo (note that it is private so you need prior access to it):

```
git clone https://github.com/thehaiwave/BabyDuck
```

Install dependencies:

```
npm i
```

Run the compiler:

```
npm run compile -i somefile.babyduck
```

The `compile` command does not take in any other options, only the `-i` flag and an input file to compile.

Optionally, you can run tests or regenerate the parser with the following commands respectively:

```
npm run test
npm run antlr
```

The tests only test the parsing. No testing is done with the resulting quadruples or interpreted in any other
way.
