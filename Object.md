## Problem

With the rise of ES2016 and functional programming in JavaScript, we tend to use frequently two methods of an Object: [Object.values](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/values) and [Object.entries](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Objets_globaux/Object/entries).

But apparently there's an issue that many of us have run through when using these methods.

Imagine that for some reason, instead of having an array containing items of a same type `T`, we have an object that contains these items (which is quite possible in case we want to get an value by referencing its key in the object). An example for this would be :

```js
/* @flow */
type Profession = {
  title: string,
  salary: number,
};
type People = { [title: string]: Profession };

const people: People = {
  Thomas: {
    title: 'Teacher',
    salary: 10000,
  },
  Kenzo: {
    title: 'Engineer',
    salary: 20000,
  },
};

const salaries: Array<number> = Object.values(people).map(person => {
  return person.salary;
});
```

We will get an error from Flow saying that:

```js
20:   return person.salary;
                    ^ property `salary`. Property cannot be accessed on
20:   return person.salary;
             ^ mixed
```

And if we want to iterate this `people` object by using :

```js
const descriptions: Array<string> = Object.entries(people).map(([name, person]) => {
  return `name : ${name}, salary: ${person.salary}`;
});
```

Flow also warns us about:

```js
24:   return `name : ${name}, salary: ${person.salary}`;
                                               ^ property `salary`. Property cannot be accessed on
24:   return `name : ${name}, salary: ${person.salary}`;
                                        ^ mixed
```

I saw some work-around examples not to use `Object.values` or `Object.entries` but the old classic method [Object.keys](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys) as follow:

```js
const details: Array<Profession> = Object.keys(people).map(name => people[name]);
```

and no Flow errors :

```js
No errors!
```

Ok so what's the problem? How do we type it correctly? As a novice JavaScript developer, it will take some time to tear the hair to find out a solution. I will explain why we got these Flow errors and why the `Object.keys` work-around isn't a truly sound solution. 

## Explanation

Here is an extract from [core.js](https://github.com/facebook/flow/blob/master/lib/core.js) of Flow project:

```js
declare class Object {
  ...
  static entries(object: any): Array<[string, mixed]>;
  ...
  static values(object: any): Array<mixed>;
}
```

Let's take a simple example. From Flow documentation about [Width Subtyping](https://flow.org/en/docs/lang/width-subtyping/), having an object `person` with the type `type Person = { name: string }` means that `person.name` has the type `string`. But it cannot prevent `person` from having other properties.

```js
/* @flow */
type Person = { name: string };
const person: Person = {
  name: 'Thierry',
  age: 20,
};
```

In this case, `Object.values(person)` will return `["Thierry", 20]` which is an array of a `string` and a `number`. And these aren't the same type which means they don't have the same properties and methods. **So that explains the extracted code above about the result as `Array<mixed>` type.**

With the same example, we would get a surprise if we expected to have an array of `string` by doing:

```js
const names: Array<string> = Object.keys(person).map(name => person[name]);
```

Flow wouldn't warn us about any error but we would possibly get one at runtime by presuming that all elements in `names` are `string` and doing some string operation as usual. As a result, **`Object.keys` isn't a truly sound work-around.**

Let's get back to our first example with the types `Profession` and `People`. Although we cannot have something like the second example before:

```js
const people: People = {
  Thomas: {
    name: 'Teacher',
    salary: 10000,
  },
  Kenzo: {
    name: 'Engineer',
    salary: 20000,
  },
  Arthur: 'Xebia',
};
```
because:

```js
17:   Arthur: 'Xebia',
              ^ string. This type is incompatible with
6: type People = { [name: string]: Profession };
                                   ^ object type
```

With the definition of Object methods above, Flow cannot be intelligent enough to figure out the exact type of `Object.values` and `Object.entries`.

## In practice

At the present (Flow version 0.57.3), the Flow team are working on it this [issue](https://github.com/facebook/flow/issues/2174#issuecomment-270214242). In practice, we have nearly two possibilities:

- Using the `Object.keys` work-around.
- Using `Object.values` or `Object.entries` + `any` casting :

```js
const salaries: Array<number> = Object.values(people).map((person: any) => person.salary);
```
- Using [map-obj](https://www.npmjs.com/package/map-obj) but once again, it introduces a more library to maintain in our codebase just for an issue of Flow.

I have a personal preference on using `any` casting even if it is not safe. But this is the only place I use `any` while waiting for a better improvement from Flow. It also helps me find all `any` easily oneday and fix all this when new version of Flow comes out.
