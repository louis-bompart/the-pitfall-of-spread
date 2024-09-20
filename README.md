# Spread+getter=:finnadie:

## Abstract
A common pattern to "extends" object and or clone them is to use the spread operator, this can lead to this kind of bizarre behavior when used with getter:


```typescript
let counter = 0;
const original = {
    get counter() {
        return counter++;
    }
};
const copy = {...original};
console.log('original.counter, expecting 0, but found:', original.counter);
// Okay, if original.counter is 1, if I call copy.counter, it should be 2, right?
console.log('copy.counter, expecting 2, but found:', copy.counter);

```

    original.counter, expecting 0, but found: 1
    copy.counter, expecting 2, but found: 0
    

## Problem

While `getter` looks like function, they are a [Property](https://developer.mozilla.org/en-US/docs/Glossary/Property/JavaScript) without `value` and instead with a `get` implementation.
Most read operation will use the `[[Get]]` [internal method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy#object_internal_methods) which will execute the `get` and use its return value as `value`.
This means that, when copy an object containing `getter` properties, those are evaluated, and the result of the evaluation is then assigned in the spread object.

## Solution

Instead of copying value, we need to copy property descriptors. ECMAScript comes with various methods to manipulate those:
* [`Object.getOwnPropertyDescriptor`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor): returns the Property Descriptor is like a Property, minus the `name`, which has been used/defined as the key given in the second parameter of the function.
* [`Object.getOwnPropertyDescriptors`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptors): returns an object akin to a map between the `name` of all Properties and their respective Property Descriptor.
* [`Object.defineProperty`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty): Set a Property Descriptor on a given key on a given object, see it as the 'set' counterpart of `Object.getOwnPropertyDescriptor`
* [`Object.defineProperties`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties): Assign all Property Descriptor of a given object, on another object, using the same keys.

With the 2nd & 4th methods, we can craft a "spread for properties" like so:


```typescript
let counter = 0;
const original = {
    get counter() {
        return counter++;
    }
};
const copy = Object.defineProperties({}, Object.getOwnPropertyDescriptors(original));
console.log('original.counter, expecting 0, found:', original.counter);
console.log('copy.counter, expecting 1, found:', copy.counter);
console.log('original.counter, expecting 2, found:', original.counter);
```

    original.counter, expecting 0, found: 0
    copy.counter, expecting 1, found: 1
    original.counter, expecting 3, found: 2
    
