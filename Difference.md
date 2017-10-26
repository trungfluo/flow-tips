## Problem

From the first reading through the documentation for [Mixed Types](https://flow.org/en/docs/types/mixed/) and [Any Types](https://flow.org/en/docs/types/any/), some of us still weren't clear on the difference between `mixed` and `any`.

In addition to that, Flow has another type named `Object` that isn't found in the actual documentation et most Javascript developers assume that this type would match all possible types in the language.

So what is the difference between them, where and when to use each of these? Especially if it happens to you to maintain a codebase that has plenty of `mixed`, `any` and `Object` pretty much everywhere.

## Explanation

From my point of view:
  - `mixed` type is simply a union of all the basic types (string, number, boolean...)
  - `any` type is a dynamic type, it can be anything and anything can be typed `any`

Let's walk through some simple examples of `mixed` and `any` to clarify that.

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

So a *mixed-typed* can't **flow into** any type variable. It is simply because a `mixed` could be a number, a boolean or something else. It doesn't has to be a number.

And what about `Object`?

`Object` is something equivalent to `{}` but unlike `mixed`, only Object can **flow into** it but not other primitive types like `string`, `number`, `boolean`, or `function`.

```js
/* @flow */
const variable_1: Object = {
  a: '1',
  b: '2',
};
const variable_2: Object = 5;
const variable_3: Object = [5];
```

We will get 
```js
6: const variable_2: Object = 5;
                              ^ number. This type is incompatible with
6: const variable_2: Object = 5;
                     ^ object type
7: const variable_3: Object = [5];
                              ^ array literal. This type is incompatible with
7: const variable_3: Object = [5];
                     ^ object type
```

## In practice

Avoid `any`, `Object` at all cost. **They are both completely unsafe.**

If a variable is treated as `any`, we will loose all advantages of Flow's type checking. Flow usually uses `any` to type the ouput of third party libraries (example: `node_modules`, if the library doesn't come from [flow-typed](https://github.com/flowtype/flow-typed)).

If a variable is treated as `Object`, Flow will allow us to access any property on it and do whatever we want with that property. For example :

```js
/* @flow */
const variable: Object = {
  a: '1',
  b: '2',
};

const number: number = variable.a;
const sum: number = variable.b + 3;
```

and Flow says :

```js
No errors!
```

I personally recommend using `mixed` to type the output of any function that we think we won't interact with its output. Good candidates for `mixed` are callback functions. Another use case I usually see is about [Using methods of Object](Object.md)