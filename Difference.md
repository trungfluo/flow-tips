## Problem

From the first reading through the documentation for [Mixed Types](https://flow.org/en/docs/types/mixed/) and [Any Types](https://flow.org/en/docs/types/any/), some of us still weren't clear on the difference between `mixed` and `any`.

In addition to that, Flow has another type named `Object` that isn't found in the actual documentation et most Javascript developers assume that this type would match all possible types in the language.

So what is the difference between them, where and when to use each of these? Especially if it happens to you to maintain a codebase that has plenty of `mixed`, `any` and `Object` pretty much everywhere.

## Explanation

From my point of view:
  - `mixed` type is simply a union of all the basic types (string, number, boolean...)
  - `any` type is a dynamic type, it can be anything and anything can be typed `any`

Let's get started with some simple examples of `mixed` and `any` to clarify that.

First:

```js
/* @flow */
const variable: number = 5;
const any_typed_variable: any = variable;
const mixed_typed_varible: mixed = variable;
```

There's no problem because a variable of some type can **flow into** a *any-typed* or *mixed-typed* variable.

Second:

```js
/* @flow */
const any_typed_variable: any = 5;
const variable: number = any_typed_variable;
```

There's no problem because a *any-typed* variable can **flow into** a number-typed variable (a boolean, string... variable are also valid in this case).

Third:

```js
/* @flow */
const mixed_type_variable: mixed = 5;
const variable: number = mixed_type_variable;
```

We get an error from Flow:
```js
3: const variable: number = mixed_type_variable;
                            ^ mixed. This type is incompatible with
3: const variable: number = mixed_type_variable;
                   ^ number
```

So a *mixed-typed* cannot **flow into** any type variable. It is simply because `mixed` could be a number, a boolean or something else. It doesn't has to be a number.

And what about `Object`?




## In practice

Avoid `any`, `Object` at all cost.