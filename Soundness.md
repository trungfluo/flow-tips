**A note on Soundness**

From my point of view, most developers found it much more difficult and time-consuming to master Flow and some of them abandoned in the middle of the path for the benefit of TypeScript.

The key difference that Flow brings into the table for me is about the conception. From [the Flow documentation](https://flow.org/en/docs/lang/types-and-expressions/):

```
In type systems, soundness is the ability for a type checker to catch every single error that might happen at runtime. This comes at the cost of sometimes catching errors that will not actually happen at runtime.

On the flip-side, completeness is the ability for a type checker to only ever catch errors that would happen at runtime. This comes at the cost of sometimes missing errors that will happen at runtime.
```

Flow's goal is to keep us away from JavaScript run-time errors and we call it `soundness`. More precisely, Flow will make worst assumptions about our code to predict possible errors we can have at runtime and sometimes these assumptions could never happen at all.

One of the first strange error that I had to cope with was something like:

```js
/* @flow */

type Collection = {
  [vehicle: string]: number | null,
};

const collection: Collection = {
  ferrari: 100000,
  mercedes: 40000,
  clio: null,
};

if (collection.ferrari) {
  console.log(collection);
  console.log(collection.ferrari.toFixed(1));
}
```

with this Flow error:

```js
15:   console.log(collection.ferrari.toFixed(1));
                  ^ call of method `toFixed`. Method cannot be called on possibly null value
15:   console.log(collection.ferrari.toFixed(1));
                  ^ null
```

I already made a type refinement by ensuring that `collection.ferrari` is truthy, why was Flow still kept saying that `collection.ferrari` might be a `null` value? I just wanted to log an object, what was the matter?

It turned out that Flow considered any function taking an array or an object in argument could mutate it when being called, which is true in JavaScript. This is the case for the first call:

```js
console.log(collection);
```

Although we all know that `console.log` just log any value passed into it and would never mutate `collection`. But wait, if instead of `console.log` we have had another API function that we didn't have any idea about its implementation, the assumption that Flow made could have been true.

So it's not only the matter of static type checker, it's also about the coding style. The best pratice here is to capture the value immediately before any function calls that could have a chance to mutate it.

```js
/* @flow */

type Collection = {
  [vehicle: string]: number | null,
};

const collection: Collection = {
  ferrari: 100000,
  mercedes: 40000,
  clio: null,
};

if (collection.ferrari) {
  const price = collection.ferrari;
  console.log(collection);
  console.log(price.toFixed(1));
}
```

The example on [Variance](Variance.md) has the same root cause but the solution is a bit different.

So that's the point why I love Flow, just like having a completely different eye that looks closely at my code to understands it and tells me that there might be an error at runtime. The more I learn to understand Flow errors like these, the more I develop a deep understanding of every line of code that I write. It's just sweet!
