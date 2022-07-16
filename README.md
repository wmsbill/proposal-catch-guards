# ECMAScript Catch Guards

This proposal adds **catch guards** to the language, enabling developers to
catch exceptions only when they match a specific pattern.

## Status

**Champions**:

- Willian Martins (he) [Netflix, [@wmsbill](https://twitter.com/wmsbill)]
- Mark Cohen (they) [Netflix, [@mpcsh\_](https://twitter.com/mpcsh_)]

**Stage**: [0](https://tc39.es/process-document)

## Problem statement

In modern JavaScript, it is possible to write conditional logic which depends
on a thrown error using `catch`, but doing so is an exercise in poor ergonomics
and error-proneness. A developer must unconditionally catch anything that might
be thrown inside a `try`, and inspect what they caught inside the `catch`:

```js
try {
  apiRequest();
} catch (err) {
  if (err instanceof InternalServerError) {
    log({ level: 'ERROR', payload: ... });
  } else if (err instanceof NotFoundError) {
    log({ level: 'INFO', payload: ... });
  } else {
    throw err;
  }
}
```

This poses two main problems:

1. It requires a superfluous extra level of nesting and the use of an extra
   conditional construct. While this may not be a problem _per se_, it
   indicates that the language could be better adapted to a very frequent use
   case. When "square peg in round hole" becomes common practice, it becomes
   worthwhile to build a square hole.
2. It requires the developer to re-throw any errors they don't want to handle.
   This process can become fraught and error-prone, and has at least [once in
   history](https://bugs.chromium.org/p/chromium/issues/detail?id=60240) caused
   misbehavior due to an engine bug.


## Terminology

A **catch guard** is a new conditional construct, designed to be used as an arm
within a `try` / `catch` block.

A **pattern**, in this proposal, should be defined according to the [pattern
matching proposal](https://github.com/tc39/proposal-pattern-matching#pattern).

## Syntax and semantics

> _Standard disclaimer: this proposal is seeking stage 1, and as such, nothing
> below represents a concrete decision or even necessarily a strong opinion. To
> that end, the prose below is intentionally informal; more concrete
> specifications will be developed before seeking stage 2._

We intend this proposal to be a follow-on to [pattern
matching](https://github.com/tc39/proposal-pattern-matching), which provides
the `match` keyword, and a grammar for patterns.

Consider the following code snippet:

```js
try {
  apiRequest();
} catch match (${InternalServerError}) {
  log({ level: 'ERROR', payload: ... });
} catch match (${NotFoundError}) {
  log({ level: 'INFO', payload: ... });
}
```

Here, we can see the benefits of this proposal at work:

- No extra nesting is required to inspect the provided error
- The familiar grammar of patterns — specifically, an [interpolation
  pattern](https://github.com/tc39/proposal-pattern-matching#interpolation-pattern)
  using the [custom matcher
  protocol](https://github.com/tc39/proposal-pattern-matching#custom-matcher-protocol)
  — provides a much more robust toolkit for crafting conditions than tools like
  `instanceof`
- Any error not caught by the specified guards continues to throw; no
  re-throwing is required

Finally, it will still be possible to chain a traditional `catch` on the end:

```js
try {
  apiRequest();
} catch match (${InternalServerError} or ${NotFoundError}) {
  retry();
} catch {
  log({ level: 'FATAL', payload: ... });
  gracefulShutdown();
}
```

Here, we can see a catch guard interacting with a traditional `catch`
statement. We think it should be possible to implement this without changing
anything about `catch` itself. Additionally, we're able to leverage [the `or`
pattern
combinator](https://github.com/tc39/proposal-pattern-matching#pattern-combinators)
to compress shared conditional logic into a single guard.
