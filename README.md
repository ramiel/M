# Memoizy

[![CircleCI](https://circleci.com/gh/ramiel/memoizy.svg?style=svg)](https://circleci.com/gh/ramiel/memoizy)

This is a memoization helper that let you memoize and also have the following features

- ⏰ Max age: discard memoized value after a configurable amount of time 
- 🗝 Custom cache key: decide how to build your cache key
- 🧹 Clear and delete: delete all the memoized values 
                    or just one for a specific argument set 
- ❓ Conditional memoization: memoize the result only if you like it. It works with async code too
- 🧪 Fully tested
- 👶 Small size, just ~50 lines of code
- 👣 Small footprint and no dependencies

## Usage

### Basic

Memoize the return value of a function

```js
const memoizy = require('memoizy');

const fact = (n) => {
  if(n === 1) return n;
  return n * fact(n - 1);
}

const memoizedFact = memoizy(fact);
memoizedFact(3); // 6
memoizedFact(3); // the return value is always 6 but
                 // the factorial is not computed anymore
```

## API

The memoize function accept the following options

`memoizy(fn, options)`

- `maxAge`: Tell how much time the value must be kept in memory, in milliseconds. Default: Infinity
- `cache`: Specify a different cache to be used. It's a function that returns a new cache that must have the same interface as [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map). Default [new Map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
- `cacheKey`: Function to build the cache key given the arguments.
- `valueAccept`: Function in the form `(err, value) => true/false`. It receive an error (if any) and the memoized value and return true/false. If false is returned, the value is discarded. If the memoized function returns a promise, the resolved value (or the rejection error) is passed to the function. Default null (all values accepted)

`memoizy` return the memoized function with two other properties: `delete` and `clear`

- `delete`: is a function that takes the same arguments as the original function and delete the entry for those arguments
- `clear`: is a function that deletes all the cached entries

## Recipes

### Expire data

```js
const memoize = require('memoizy');

const double = memoize(a => a * 2, {maxAge: 2000});

double(2); // 4
double(2); // returns memoized 4
// wait 2 seconds, memoized value has been discarded
double(2); // Original function is called again and 4 is returned. The value is memoized for other 2 seconds
```

### Discard rejected promises

```js
const memoize = require('memoizy');

const originalFn = async (a) => {
  if(a > 10) return 100;
  throw new Error('Value must be more then 10');
}
const memoized = memoize(originalFn, {valueAccept: (err, value) => !err});

await memoized(1); // throw an error and the value is not memoized
await memoized(15); // returns 100 and the value is memoized
```


### Discard some values

```js
const memoize = require('memoizy');

const originalFn = (a) => {
  if(a > 10) return true;
  return false;
}
// Tell to ignore the false value returned
const memoized = memoize(originalFn, {valueAccept: (err, value) => value === true});

await memoized(1); // ignores the result since it's false
await memoized(15); // returns true and it's memoized
```

### Delete and Clear

```js
const memoize = require('memoizy');

const sum = (a, b) => a + b;

const memSum = memoize(sum);

memSum(5, 4); // returns 9;
memSum(1, 3); // returns 4;
memSum.delete(5,4); // remove the entry for the memoized result 9
memSum(1, 3); // returns 4 without comupting the sum
memSum(5, 4); // returns 9 re-computing the sum and memoize it again

memSum.clear(); // All values are now cleared and the cache for this memoized function is empty
```

### Different cache

You can use another cache implementation if you desire. The only contraints is that it must implement
the methods `has`, `get`, `set`, `delete` and `clear`.    

**NOTE**: If you plan to use a WeakMap, remember that clear is not available in all the implementations
Look [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap#Implementing_a_WeakMap-like_class_with_a_.clear()_method=) for a way to use a weak map as cache.

```js
const memoizy = require('memoizy');

class AlternativeCache {
  constructor() {
    this.data = {};
  }

  has(key) {
    return key in this.data;
  }

  get(key) {
    return this.data[key];
  }

  set(key, value) {
    this.data[key] = value;
  }

  delete(key) {
    delete this.data[key];
  }

  clear() {
    this.data = {};
  }
}
const fn = a => a * 2;
const memFn = memoizy(fn, {cache: () => new AlternativeCache()});
```
