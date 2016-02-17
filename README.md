# Object.keysIn / Object.valuesIn / Object.entriesIn

## Rationale

Following the [rationale](https://github.com/tc39/proposal-object-values-entries#rationale) in [Object.values and Object.entries](https://github.com/tc39/proposal-object-values-entries) proposal, all these methods are useful to obtain an array of keys, values, and key/value pairs (what the spec calls “entries”) from an object, for the purposes of iteration or serialization.

They are helpful for handling Map and Set objects and are also largely used on libraries like LoDash. The name follows LoDash's `_.keysIn` method and JS `Object.{keys, values, entries}`.

The return is similar to current triplet `{keys, values, entries}` in Object, the main difference is the enumerating the properties in the prototype chain, like the `for in` loop. Object.keysIn has got an extra motivation after `Reflect.enumerate` removal, following the removal of the `[[Enumerate]]` internal method.

With a spread use of prototypal chain in objects, the serialization of own and inherited entries is valuable. The large use adoption of LoDash methods is a good use case.

## Examples

```js
// Given the current variables:
var results;
var iterSuper = {
  foo: 42
};
var iter = Object.create( iterSuper );
iter.bar = 43;
```

### `Object.keysIn( iter )`

```js
// Before
results = [];

for ( let x in iter ) {
  results.push( x );
}

results; // [ "foo", "bar" ]

// After
results = Object.keysIn( iter );
results; // [ "foo", "bar" ] (same order as for loop)
```



### `Object.valuesIn( iter )`

```js
// Before
results = [];

for ( let x in iter ) {
  results.push( iter[ x ] );
}

results; // [ 42, 43 ]

// After
results = Object.valuesIn( iter );
results; // [ 42, 43 ] (same order as for loop)
```



### `Object.entriesIn( iter )`

```js
// Before
results = [];

for ( let x in iter ) {
  results.push( [ x, iter[ x ] ] );
}

results; // [ [ "foo", 42 ], [ "bar", 43 ] ]

// After
results = Object.entriesIn( iter );
results; // [ [ "foo", 42 ], [ "bar", 43 ] ] (same order as for loop)
```
