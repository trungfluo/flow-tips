## Problem

As the time of writing this article, there's a ton of others that explain clearly what are variances (read-only or write only) problems in Flow. I found that two of them are excellent:

- [Flow’s best kept secret](https://medium.com/@forbeslindesay/covariance-and-contravariance-c3b43d805611)
- [Property Variance and Other Upcoming Changes](https://flow.org/blog/2016/10/04/Property-Variance/)

I recommend that every of us should read these two. If you understand these two, you could skip this article from now. I think I would simplify more the problem exposed by these articles and insist on a more simplified solution.

Suppose that we have a string logging method that takes an object as its input:

```js
/* @flow */
function logNullableString(obj: { value: ?string }): void {
  console.log(obj.value);
}

const a: { value: string } = {
  value: 'hey',
};

logNullableString(a);
```

When we run Flow, we get a very confused message:

```js
24: logNullableString(a);
                      ^ object type. This type is incompatible with the expected param type of
16: function logNullableString(obj: { value: ?string }): void {
                                    ^ object type
Property `value` is incompatible:
16: function logNullableString(obj: { value: ?string }): void {
                                             ^ null. This type is incompatible with
20: const a: { value: string } = {
                      ^ string
24: logNullableString(a);
                      ^ object type. This type is incompatible with the expected param type of
16: function logNullableString(obj: { value: ?string }): void {
                                    ^ object type
Property `value` is incompatible:
16: function logNullableString(obj: { value: ?string }): void {
                                             ^ undefined. This type is incompatible with
20: const a: { value: string } = {
                      ^ string
```

The same problem occurs with `Array`:

```js
function logNullableValuedArray(arr: Array<?string>): void {
  arr.forEach(item => {
    console.log(item);
  });
}

const b: Array<string> = ['bye'];

logNullableValuedArray(b);
```

with these horrible errors: 

```js
6: logNullableValuedArray(b);
                           ^ array type. This type is incompatible with the expected param type of
28: function logNullableValuedArray(arr: Array<?string>): void {
                                         ^ array type
Type argument `T` is incompatible:
28: function logNullableValuedArray(arr: Array<?string>): void {
                                               ^ null. This type is incompatible with
34: const b: Array<string> = ['bye'];
                   ^ string
36: logNullableValuedArray(b);
                           ^ array type. This type is incompatible with the expected param type of
28: function logNullableValuedArray(arr: Array<?string>): void {
                                         ^ array type
Type argument `T` is incompatible:
28: function logNullableValuedArray(arr: Array<?string>): void {
                                               ^ undefined. This type is incompatible with
34: const b: Array<string> = ['bye'];
                   ^ string
```

It is to note that these codes won't throw any errors at runtime even if we got warnings from Flow.

## Explanation

The problem with these codes is that `object` or `array` are `mutable`, so `logNullableString` or `logNullableValuedArray` functions could legally do

```js
obj.value = null;
arr.push(null);
```

If Flow allowed this subtyping relationship, then the original object (`a`), that expects it had an attribute whose value had type `string`, would now have one that is `null`, thereby breaking the type system. The same for `b` that expects it contained only element of type `string`, would now have one that is `null`.

## In practice

To prevent Flow from thinking that we could `mutate` the object inside the function, we use `+` for the property of the object in question, and `$ReadOnlyArray` in case of Array.

These examples above would be done as follow:

```js
/* @flow */
function logNullableString(obj: { +value: ?string }): void {
  console.log(obj.value);
}

const a: { value: string } = {
  value: 'hey',
};

logNullableString(a);

function logNullableValuedArray(arr: $ReadOnlyArray<?string>): void {
  arr.forEach(item => {
    console.log(item);
  });
}

const b: Array<string> = ['bye'];

logNullableValuedArray(b);
```

In practice, I usually see that this problem occurs with union types but the nature of the question is still the same. 