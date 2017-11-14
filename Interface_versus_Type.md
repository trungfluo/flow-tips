## Problem

Lots of JS developers who actually use Flow have some TypeScript backgrounds and I believe that some of them got confused with the keywords [interface](https://flow.org/en/docs/types/interfaces/) and [type](https://flow.org/en/docs/types/objects/) at the beginning. Both these keywords are valid in both Flow and Typescript. 

So supposed we have

```js
/* @flow */
type SerializableObj = {
  serialize(): string
};

interface Serializable {
  serialize(): string;
};
```

These declarations **seem to be the same thing**. In simple cases one can be used interchangeably with the other. So how do we choose `type` or `interface`? 

In the scope this post I try to explain some little differences between `interface` and `type` **in Flow**.

## Explanation

From the [docs](https://flow.org/en/docs/types/interfaces/), if we change the official example:

```js
interface Serializable {
  serialize(): string;
}

class Foo implements Serializable {
  serialize() { return '[Foo]'; }
}
```

to 

```js
type Serializable {
  serialize(): string;
}

class Foo implements Serializable {
  serialize() { return '[Foo]'; }
}
```

we get some errors:

```js
3: type Serializable {                     ^ Unexpected token {
4:   serialize(): string;
              ^ Unexpected token (
4:   serialize(): string;                        ^ Unexpected token ;
5: }   ^ Unexpected token }
7: class Foo implements Serializable {
   ^ Unexpected token class
7: class Foo implements Serializable {
             ^ Use of future reserved word in strict mode
7: class Foo implements Serializable {
                        ^ Unexpected identifier
7: class Foo implements Serializable {                                     ^ Unexpected token {
8:   serialize() { return '[Foo]'; } // Works!
                 ^ Unexpected token {
8:   serialize() { return '[Foo]'; } // Works!
                   ^ Illegal return statement
```

So the major difference is if we want to use `implements` for ES2015 Class, we must use an `interface`, not a `type`.

Again from the [docs](https://flow.org/en/docs/types/interfaces/):

```
Classes in Flow are nominally typed. This means that when you have two separate classes you cannot use one in place of the other even when they have the same exact properties and methods.
...
you can use interface in order to declare the structure of the class that you are expecting.
```

Using `implements` for ES2015 isn't necessarily mandatory to guarantee the structure of a class instance. We can give to the last one with a compatible `type` that has the expected structure:

```js
type Serializable = {
  serialize(): string;
}

class Foo {
  serialize() { return '[Foo]'; }
}

const foo: Serializable = new Foo();
```

The last note that I would like to call our attention is about the difference between two following declarations:

```js
interface Serializable_Method {
  serialize(): string;
}

interface Serializable_Attribute {
  serialize: () => string;
}
```

By default, any methods declared on an interface in Flow is **read-only** because all these methods will be implemented in classes that `implements` this interface. This is the first declaration. 

The second declaration indicates that `serialize` is a property of `Serializable_Attribute`. Again from the [docs](https://flow.org/en/docs/types/interfaces/):

```
Interface properties are invariant by default.
```

So if we have

```js
class Foo implements Serializable_Method {
  serialize() {
    return '[Foo]';
  }
}

const foo: Serializable_Attribute = new Foo();
```

We will get a pretty clear error message from Flow 

```js
15: const foo: Serializable_Attribute = new Foo();
                                        ^ Foo. This type is incompatible with
15: const foo: Serializable_Attribute = new Foo();
               ^ Serializable_Attribute
Property `serialize` is incompatible:
15: const foo: Serializable_Attribute = new Foo();
                                        ^ Foo. Covariant property `serialize` incompatible with invariant use in
15: const foo: Serializable_Attribute = new Foo();
               ^ Serializable_Attribute
```

I tried to replace these declarations by `type` instead of `interface` and I got the same error message.

## In practice

I suppose that Flow team is trying to unify these two concepts `type` and `interface` (as showed in this discussion [link](https://stackoverflow.com/questions/36904201/when-do-you-use-an-interface-over-a-type-alias-in-flow)). With that link, I extract the best pratices for using `type` and `interface`:

```
- Use object types to describe bags of mostly data that are passed around in your app, e.g., props/state for React components, Flux/Redux actions, JSON-like stuff.

- Use interfaces to describe service-like interfaces. Usually these are mostly methods, e.g., Rx.Observable/Observer, Flux/Redux stores, abstract interfaces. If a class instance is likely to be an inhabitant of your type, you probably want an interface.
```

In practice, I usually use `interface` inside the [library definition](https://flow.org/en/docs/libdefs/) to define third-party libraries on my own.
