# jest-import-equality-issue

Reproducible example for Jest issue

## Description

Jest has an option (`resetModules`) to reset module imports between tests. When using dynamic `require` statements in tested code, the required object will not be referencially equal to objects imported from the same module using static `require`.

For example:

```js
const MyClass = require('./MyClass')

module.exports = () => MyClass === require('./MyClass')
// These objects are not equal with resetModules=true
```

### Theoretical Cause

Modules imported with `require` are cached. When using dynamic imports Jest invokes `require` while executing the tested code, but this uses a separate cache to those imports determined when the subject module was imported into the test file.

## Reproducible Setup

```sh
git clone https://github.com/atkinchris/jest-import-equality-issue.git

cd jest-import-equality-issue

yarn install # or 'npm install'
```

### Behaviour in Node

This should log `true` as the equality is maintained in Node.

```sh
$ node -e "console.log(require('./src/example')())"
true
```

### Passing Behaviour

This runs the test with Jest's config option `resetModules` set to `false`. The test should pass.

```sh
$ jest --config=withoutReset.config.js
 PASS  src/example.test.js
  ✓ should be "true" (5ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        1.116s
Ran all test suites.
✨  Done in 1.75s.
```

### Failing Behaviour

This runs the test with Jest's config option `resetModules` set to `true`. The test should fail.

```sh
$ jest --config=withReset.config.js
 FAIL  src/example.test.js
  ✕ should be "true" (10ms)

  ● should be "true"

    expect(received).toBe(expected) // Object.is equality

    Expected: true
    Received: false

      3 | it('should be "true"', () => {
      4 |   const result = example()
    > 5 |   expect(result).toBe(true)
        |                  ^
      6 | })
      7 |

      at Object.toBe (src/example.test.js:5:18)

Test Suites: 1 failed, 1 total
Tests:       1 failed, 1 total
Snapshots:   0 total
Time:        0.944s, estimated 1s
Ran all test suites.
```
