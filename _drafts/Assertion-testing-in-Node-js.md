title: "Assertion testing in Node.js"
tags:
---
Node.js provides a module for writing assertions and writing some simple unit tests. It has some basic functions for writing useful tests on your code.

## Code Walkthrough

### assert.fail(actual, expected, message, operator)

Throws an exception

### assert(value[, message]), assert.ok(value[, message])
### assert.equal(actual, expected[, message])
### assert.notEqual(actual, expected[, message])
### assert.deepEqual(actual, expected[, message])
### assert.notDeepEqual(actual, expected[, message])
### assert.strictEqual(actual, expected[, message])
### assert.notStrictEqual(actual, expected[, message])
### assert.throws(block[, error][, message])
### assert.doesNotThrow(block[, message])
### assert.ifError(value)