## Problem

With the rise of ES2016 and functional programming in JavaScript, we tend to use frequently two methods of an Object: [Object.values](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Objets_globaux/Object/values) and [Object.entries](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Objets_globaux/Object/entries).

But apparently there's an issue that many of us have run through when using these methods.

Imagine that for some reason, instead of having an array containing items of a same type `T`, we have an object that contains these items (which is quite possible in case we want to get an value by referencing its key in the object). An example for this would be :

```js
/* @flow */
type Profession = {
  name: string,
  salary: number,
};
type People = { [name: string]: Profession };

const people: People = {
  Thomas: {
    name: 'Teacher',
    salary: 10000,
  },
  Kenzo: {
    name: 'Engineer',
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

So what's the problem? How do we type it correctly? As a novice JavaScript developer, it will take some time to tear the hair before finding out a sound solution.

## Explanation


```js
declare class Object {
  ...
  static entries(object: any): Array<[string, mixed]>;
  static values(object: any): Array<mixed>;
}
```


## In practice