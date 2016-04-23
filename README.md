# Object.keysIn / Object.valuesIn / Object.entriesIn

## Rationale

Following the [rationale](https://github.com/tc39/proposal-object-values-entries#rationale)
in [Object.values and Object.entries](https://github.com/tc39/proposal-object-values-entries)
proposal, all these methods are useful to obtain an array of keys, values, and
key/value pairs (what the spec calls “entries”) from an object, for the purposes
of iteration or serialization.

They are largely used by libraries like Lodash. The method names are based on Lodash’s `_.keysIn`,
`_.valuesIn`, and `_.forIn` methods, which were inspired by
[for…in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in),
and JS `Object.{keys, values, entries}`.

The return is similar to the current triplet `{keys, values, entries}` in `Object`,
the main difference is the enumerating the properties in the prototype chain, like
the `for…in` loop. `Object.keysIn` has got an extra motivation after
`Reflect.enumerate` removal, following the removal of the `[[Enumerate]]`
internal method.

The current methods are consistent with `Reflect.enumerate` and the `for…in` loop
as it only list String-valued keys of enumerable properties, ignoring any symbol
keys.

## array vs iterator

These additions are complementary to the existing `Object.keys`, `Object.values`,
and `Object.entries`. They are basically the same methods including inherited
String-value keys.

## Use cases

With a spread use of prototypal chain in objects, the serialization of own and
inherited entries is valuable. The large use adoption of Lodash methods is a
good use case.

jQuery's [`$.extend`](http://api.jquery.com/jQuery.extend/) and
[`$.fn.extend`](http://api.jquery.com/jQuery.fn.extend/) and Underscore/Lodash
[`_.extend`/`_.assignIn`](https://lodash.com/docs#assignIn) and
[`_.defaults`](https://lodash.com/docs#defaults) iterate over own and inherited
enumerable properties. In the case of Lodash, [`keysIn`](https://lodash.com/docs#keysIn)
is used to get the property names to iterate over. It's also used in Underscore
`_.functions` and Lodash's [`_.functionsIn`](https://lodash.com/docs#functionsIn).

The use of `Object.valuesIn` can be mapped to the use of `values` which is used
in library helpers like Underscore/Lodash [`includes`](https://lodash.com/docs#includes)
for seeing if a value is in an object or [`sample`](https://lodash.com/docs#sample)
which grabs a random value from an object.

Node.js DB packages like [Mongoose](https://github.com/Automattic/mongoose) use
some of these Lodash methods when working with data objects.

Lodash's [`_.toPlainObject`](https://lodash.com/docs#toPlainObject) which
flattens inherited properties to own properties of a new object. This is handy
for working with `Object.assign` and `keysIn` is used to implement that.

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

### `Object.keysIn( O )`

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

### `Object.valuesIn( O )`

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

### `Object.entriesIn( O )`

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

----

## Spec

### Object.keysIn( O )

1. Let _obj_ be ? ToObject(_O_).
1. Let _nameList_ be ? EnumerableProperties(_obj_, __"key"__).
1. Return CreateArrayFromList(_nameList_).

### Object.valuesIn( O )

1. Let _obj_ be ? ToObject(_O_).
1. Let _nameList_ be ? EnumerableProperties(_obj_, __"value"__).
1. Return CreateArrayFromList(_nameList_).

### Object.entriesIn( O )

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
