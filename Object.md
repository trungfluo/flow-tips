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

Object.values(people).map(person => {
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
Object.entries(people).map(([name, person]) => {
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

Ok so what's the problem? How do we type it correctly? As a novice JavaScript developer, it will take some time to tear the hair before finding out a truly sound solution. I will explain why we got these Flow errors and why not to use the `Object.keys` work-around. 

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

From Flow documentation about [Width Subtyping](https://flow.org/en/docs/lang/width-subtyping/), having an object `person` with the type `type Person = { name: string }` means that `person.name` has the type `string`. But it cannot prevent `person` from having other properties.

```js
/* @flow */
type Person = { name: string };
const person: Person = {
  name: 'Thierry',
  age: 20,
};
```

In this case, `Object.values(person)` will return `["Thierry", 20]` which is an array of a `string` and a `number`. And these don't have the same properties and methods. So that explains the extracted code above.

## In practice