## Problem

According to [Wikipedia](https://en.wikipedia.org/wiki/Result_type):

```
In functional programming, a result type is a Monadic type holding a returned value or an error code. They provide an elegant way of handling errors, without resorting to exception handling; when a function that may fail returns a result type, the programmer is forced to consider success or failure paths, before getting access to the expected result; this eliminates the possibility of an erroneous programmer assumption.
```

Unfortunately this isn't a default feature of JavaScript (which wasn't designed to be functional) and it becomes a very common pattern for developers to implement in the language.

## In practice

With Flow it becomes easy to represent a result type by using Enum type and generic type. As discussed in [If Statement](IfStatement.md) section, we will try to have a common property inside each success/error type to make the pattern matching easier.

```js
/* flow */
export type SuccessType<S> = S & {
  ok: true,
};

export type ErrorType<E> = {
  ok: false,
  error: E,
};

export type Result<S, E> = SuccessType<S> | ErrorType<E>;
```

I think it is enough with these three types for all cases we need to handle. But having to build objects for these types all the times might be painful. We can have some default builder functions to handle that:

```js
export function succeed<T>(object: T): SuccessType<T> {
  return {
    ok: true,
    ...object,
  };
}

export function fail<T>(error: T): ErrorType<T> {
  return {
    ok: false,
    error: error,
  };
}
```
