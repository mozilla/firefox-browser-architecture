---
title: JavaScript Type Safety Systems
layout: text
---

# JavaScript Type Safety Systems

JavaScript is an untyped language and as such is susceptible to type safety bugs
where a type other than that expected is used by mistake. For example trying
to add a number to a string and expecting a numeric result.

JavaScript type safety systems work as an extra layer of code syntax on top of
the raw JavaScript. Some kind of compiler or checker is used to verify that
type assertions made in the code are met allowing bugs to be detected.

Recent studies have suggested that using type annotations in JavaScript can stop
as many as 10% of bugs from entering the product.

## Flow

Flow is a type safety system developed by Facebook. When no types are provided
in the source code it attempts to infer types based on variable assignments.
Flow comes in two forms. The more common form involves adding types as an
extended JavaScript syntax. This form must be removed by a transpiler at build
time. The other forms are comment based, one only supports function argument
typing the other supports any typing but is IMO quite ugly.

Flow frequently finds bugs that aren’t errors and so would need work to correct
before any typing can be used with the file. As an example in a file with 4k
lines of code that we can assume to be largely bug free Flow found some 122
errors so one error per 32 lines. Two of them were potential bugs (though
actually inconsequential). The rest were either Flow bugs, cases where Flow was
pointing out a potential issue that could never occur in reality or cases where
we use complications that Flow can’t reasonably be expected to understand.

## TypeScript

TypeScript is a typed superset of JavaScript that compiles to plain JavaScript.
TypeScript files support compiling to an older version of JavaScript supported
than the version written. TypeScript does some type inference so some validation
is performed even before any types are added but less than Flow.

When run on an existing large file in the repository TypeScript found around one
issue per hundred lines of code which is less than Flow found on the same file.
Most of those issues were misunderstood types. None of them were errors that
would cause problems in the code.

## Conclusions

In the case of both Flow and TypeScript significant work would need to be done
in order to use them in existing code. Both suffer from a number of drawbacks:

* Not understanding some modern JS syntaxes
* Not understanding our module system
* Requiring build-time processing leading to difficulties with logging and debugging

It doesn't seem worth the work at this stage to use either system in the main
Mozilla codebase.
