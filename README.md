# Object.mapKeys / Object.mapValues / Object.mapEntries

## Rationale

Following the [rationale](https://github.com/tc39/proposal-object-values-entries#rationale)
in [Object.values and Object.entries](https://github.com/tc39/proposal-object-values-entries)
proposal, all these methods are useful to obtain an array of keys, values, and
key/value pairs (what the spec calls “entries”) from an object, for the purposes
of iteration or serialization.

They are helpful for handling `Map` and `Set` objects and are also largely used
by libraries like Lodash. The method names are based on Lodash’s `_.keysIn`,
`_.valuesIn`, and `_.forIn` methods, which were inspired by
[for…in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in),
and JS `Object.{keys, values, entries}`.

The return is similar to the current triplet `{keys, values, entries}` in `Object`,
the main difference is the enumerating the properties in the prototype chain, like
the `for…in` loop. `Object.mapKeys` has got an extra motivation after
`Reflect.enumerate` removal, following the removal of the `[[Enumerate]]`
internal method.

With a spread use of prototypal chain in objects, the serialization of own and
inherited entries is valuable. The large use adoption of Lodash methods is a
good use case.

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

### `Object.mapKeys( O )`

```js
// Before
results = [];

for ( let x in iter ) {
  results.push( x );
}

results; // [ "foo", "bar" ]

// After
results = Object.mapKeys( iter );
results; // [ "foo", "bar" ] (same order as for loop)
```

### `Object.mapValues( O )`

```js
// Before
results = [];

for ( let x in iter ) {
  results.push( iter[ x ] );
}

results; // [ 42, 43 ]

// After
results = Object.mapValues( iter );
results; // [ 42, 43 ] (same order as for loop)
```

### `Object.mapEntries( O )`

```js
// Before
results = [];

for ( let x in iter ) {
  results.push( [ x, iter[ x ] ] );
}

results; // [ [ "foo", 42 ], [ "bar", 43 ] ]

// After
results = Object.mapEntries( iter );
results; // [ [ "foo", 42 ], [ "bar", 43 ] ] (same order as for loop)
```

----

## Spec

### Object.mapKeys( O )

1. Let _obj_ be ? ToObject(_O_).
1. Let _nameList_ be ? EnumerableProperties(_obj_, __"key"__).
1. Return CreateArrayFromList(_nameList_).

### Object.mapValues( O )

1. Let _obj_ be ? ToObject(_O_).
1. Let _nameList_ be ? EnumerableProperties(_obj_, __"value"__).
1. Return CreateArrayFromList(_nameList_).

### Object.mapEntries( O )

1. Let _obj_ be ? ToObject(_O_).
1. Let _nameList_ be ? EnumerableProperties(_obj_, __"key+value"__).
1. Return CreateArrayFromList(_nameList_).

### EnumerableProperties

When the abstract operation EnumerableProperties is called with Object _O_ and
String _kind_ the following steps are taken:

1. Assert: Type(_O_) is Object.
1. Let _iterator_ be ? EnumerateObjectProperties(_obj_).
1. Let _properties_ be a new empty list.
1. Repeat
  1. Let _next_ be ? IteratorStep(_iterator_)
  1. If _next_ is __false__, return _properties_.
  1. Let _nextArg_ be ? IteratorValue(_next_).
  1. If _kind_ is __"key"__, append _nextArg_ as the last element of _properties_.
  1. Else,
    1. Let _value_ be ? Get(_O_, _key_).
    1. If _kind_ is __"value"__, append _value_ to _properties_.
    1. Else,
      1. Assert: _kind_ is __"key+value"__.
      1. Let _entry_ be CreateArrayFromList(« _key_, _value_ »).
      1. Append _entry_ to _properties_.
