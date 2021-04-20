# ECMAScript Catch Guards

This proposal adds catch guards to the language, enabling developers to catch
exceptions only when they match a specific class or classes.

This proposal complements the [pattern matching proposal
](https://github.com/tc39/proposal-pattern-matching) and does not try to
compete with it.

This proposal draws heavily from similar features available in
[Java](https://docs.oracle.com/javase/specs/jls/se7/html/jls-14.html#jls-14.20),
[C#](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/try-catch),
[Ruby](http://rubylearning.com/satishtalim/ruby_exceptions.html), and
[Python](https://docs.python.org/3/tutorial/errors.html#handling-exceptions).

## Status

**Champions**: Guilherme Hermeto (Netflix, [@ghermeto](https://twitter.com/ghermeto))

**Stage**: 0

**Lastest update**: draft created

## Motivation

- **Developer ergonomics**:
  
  Today, developers have to catch all Errors, add a `if` or `switch` statement and 
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

  Engines will be able to skip completelly the blocks if the error doesn't matches
  the correct type.
  
## Syntax

### Option 1 (using `as`):

```javascript
class ConflictError extends Error {}
class NotFoundError extends Error {}
class OtherError extends Error {}
  
try {
  something();
} catch (ConflictError as conflict) {
  // handle it one way...
} catch(NotFoundError | OtherError as other) {
  // handle the other way...
} catch (err) {
  // catch all...
}
```

or:

```javascript
class ConflictError extends Error {}
class NotFoundError extends Error {}
class OtherError extends Error {}
  
try {
  something();
} catch (^ConflictError as conflict) {
  // handle it one way...
} catch(^NotFoundError | ^OtherError as other) {
  // handle the other way...
} catch (err) {
  // catch all...
}
```



### Option 2 (using `if instanceOf`):

```javascript
class ConflictError extends Error {}
class NotFoundError extends Error {}
class OtherError extends Error {}
  
try {
  something();
} catch (conflict if instanceOf ConflictError) {
  // handle it one way...
} catch(other if instanceOf NotFoundError | OtherError) {
  // handle the other way...
} catch (err) {
  // catch all...
}
```

### Option 3 (using `:`):

```javascript
class ConflictError extends Error {}
class NotFoundError extends Error {}
class OtherError extends Error {}
  
try {
  something();
} catch (conflict: ConflictError) {
  // handle it one way...
} catch(other: NotFoundError | OtherError) {
  // handle the other way...
} catch (err) {
  // catch all...
}
```

## Implementations

* Babel Plugin //TBD

## Q&A

#### What if the error is an string?

We are trying to solve for types only, so this would work:

```javascript
try {
  something();
} catch (String as err) {
  // handle it...
}
```

But it would catch all errors thrown as strings. Hopefully the
[pattern matching proposal](https://github.com/tc39/proposal-pattern-matching)
will address the use-case of matching regular expressions.
