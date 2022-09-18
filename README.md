# ECMAScript Catch Guards

This proposal adds the catch guards to the language, enabling developers to catch
exceptions only when they match a specific class or set of classes.

This proposal draws heavily from similar features available in
[Java](https://docs.oracle.com/javase/specs/jls/se7/html/jls-14.html#jls-14.20),
[C#](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/try-catch),
[Ruby](http://rubylearning.com/satishtalim/ruby_exceptions.html), and
[Python](https://docs.python.org/3/tutorial/errors.html#handling-exceptions).

## Problem statement
Javascript doesn't provide an ergonomic way of treating different types of errors.

## Status

**Champion**: 
Willian Martins (Netflix, [@wmsbill](https://twitter.com/wmsbill)),


**Stage**: 0

**Lastest update**: draft created

## Motivation

- **Developer ergonomics**:
  
  Today, developers have to catch all Errors, add an `if` or `switch` statement and 
  rethrow even when they want to catch one particular type of error:

  ```javascript
  class ConflictError extends Error {}
  
  try {
    something();
  } catch (err) {
    if (err instanceOf ConflictError) {
      // handle it...
    }
    throw err;
  }
  ```
  
- **Parity with other languages**:
  
  Scoping error handling to specific types is a common construct in several programming
  languges, as:
  [Java](https://docs.oracle.com/javase/specs/jls/se7/html/jls-14.html#jls-14.20),
  [C#](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/try-catch),
  [Ruby](http://rubylearning.com/satishtalim/ruby_exceptions.html), and
  [Python](https://docs.python.org/3/tutorial/errors.html#handling-exceptions).

  
- **Code/Engine Optimization**:

Engines can skip the block entirely if the error doesn't match the correct type.
  
## Syntax options

The current implementation uses the following syntax:
```
 catch ( CatchParameter[?Yield, ?Await] ) { block }
```

Where `CatchParameter` is:
* `BindingIdentifier[?Yield, ?Await]`
* `BindingPattern`


### Option 1 (New Catch Statement):

This proposal introduces a new Binding type for the CatchParameter called `BindingGuard`; this new bind type's shape and semantics are subject to stage 1 development.

```javascript
  
try {
  something();
} catch (BindingGuard) {
  // handle it one way...
} catch(BindingGuard) {
  // handle the other way...
} catch (err) {
  // catch all...
}
```

### Option 1 bis (New Catch Statement with optionnal type):

This proposal introduces a specific syntax that imply that ErrorType is an Optionnal argument and when given is used to catch error with specific type.

```javascript  
try {
  something();
} catch (err, ErrorType) {
  // handle it one way...
} catch(err, ErrorType) {
  // handle the other way...
} catch (err) {
  // catch all...
}
```

transpile to

```javascript  
try {
  something();
} catch (err) {
  if(err instanceof ErrorType) {
    // handle it one way...
  } else if(err instanceof ErrorType) {
    // handle the other way...
  } else {
    // catch all...
  }
}
```

or this

```javascript  
try {
  something();
} catch (err, ErrorType) {
  // handle it one way...
}
```

transpile to

```javascript  
try {
  something();
} catch(err) {
  if(err instanceof ErrorType){
    // handle it one way...
  } else throw err
}
```

### Option 2 (New CatchPatten statement):

This option can be used in conjunction with option one or leveraged on the Pattern Matching proposal to bind different error patterns (syntax and semantics are subject to stage 1 development), making the catch statement keep its current syntax.


```javascript
  
try {
  something();
} CatchPatten (BindingGuard) {
  // handle it one way...
} CatchPatten (BindingGuard) {
  // handle the other way...
} catch (err) {
  // catch all...
}
```

## Implementations

* Babel Plugin //TBD

