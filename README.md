[![Build Status](https://travis-ci.org/joshvillbrandt/localForageLRU.svg?branch=master)](https://travis-ci.org/joshvillbrandt/localForageLRU)

# localForageLRU

A wrapper around `localForage` that provides an LRU cache behavior.

## Background

This library seeks to provide a local storage interface for web browsers that can tolerate very large database sizes. A least recently used (LRU) policy is implemented on top of [localForage](https://github.com/mozilla/localForage). The API for `localForageLRU` is exactly the same as it is for `localForage` - the only difference is that old objects are removed as necessary to put in new objects.

## Setup

Install with bower:

```bash
bower install localforagelru --save
```

## Usage

Good news, everyone! The API for `localForageLRU` is exactly the same as it is for `localForage` - simply use the `localforagelru` global instead of the `localforage` global. Callback and promise patterns are both supported. Check out the [localForage README](https://github.com/mozilla/localForage) and [localForage API docs](http://mozilla.github.io/localForage/) for detailed API information.

Here is an example using the promise pattern:

```javascript
// A full setItem() call with Promises.
localforagelru.setItem('key', 'value').then(function(value) {
    alert(value + ' was set!');
}, function(error) {
    console.error(error);
});
```

Behind the scenes, `localForageLRU` listens for "over quota" errors and removes least recently used items from the data store until the requested object can fit into the data store. In addition to the standard `localForage` configuration options, `localForageLRU` supports two additional options:

* recencyKey - this is used as the key to store the recency list; defaults to `_localforagelru_recency`
* debugLRU - set this to `true` to see when objects get kicked out of the store; defaults to `false`

This library also ships with a stress tester so that you can see the cache in action. First try the stress tester with a normal `localforage` instance to see the `QuotaExceededError` errors, then try it with a `localforagelru` instance to see the caching pruning in action. On my browser, my IndexedDB cache holds nearly 5.5 GB before throwing a `QuotaExceededError` error.

```javascript
// api:
// localforageStressTester = function(store, endKey, startKey)

// clear out the DB first (wait for console output before preceding)
localforage.clear().then(function() { console.log('Store has been cleared!'); });

// try this first to make sure everything is working fine
localforageStressTester(localforage, 10); // should see: (10) setItem success
localforagePrintLength(localforage); // should see length of 11 (the recency list is the extra one)

// test with plain localforage
localforageStressTester(localforage, 6000); // expecting to see a quota error at some point

// test again with localforagelru
// hint: to save time, set startKey to something just before where the last test failed
var localforagelrudebug = localforagelru.createInstance({debugLRU: true});
localforageStressTester(localforagelrudebug, 6000, 5000); // expecting to see "item removed" notifications
```

## Development

This library has been developed with node 0.10. If you don't have node, first install it with [nvm](https://github.com/creationix/nvm).

Here is how to checkout the code, install the dependencies, and run the tests:

```bash
git checkout https://github.com/joshvillbrandt/localForageLRU.git
cd localForageLRU
npm install
npm test
```

## Change Log

This project uses [semantic versioning](http://semver.org/).

### 1.1.0 - 2015/09/07

* Fix package name for bower case sensitivity
* Remove in-memory of copy of recency list to concurrency errors because of multiple clients
* Fix debug console error and check specifically for quota failure
* Add `localforageStressTester()`

### 1.0.0 - 2015/09/06

* Initial release
