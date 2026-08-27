# Browser Test

Don't want to bother setting up a browser automation framework to run
a few tests? Just drop `browser-test.html` into your project and start
writing.

If you find yourself writing javascript that needs access to browser
apis not available on server-side JS engines, you're faced with a hard
choice:

1. Mock out the relevant API. This may be very challenging based on
   the complexity of the API, and it will be extra difficult that the
   mock is accurate enough to adequately test your code.
2. Setup browser automation with Selenium / Playwrite / Cypress or
   other tools. Heavyweight, hard to configure, slow, these tools
   work, but are not fun or easy to use. They also often move you up
   the ladder from unit-tests to integration tests.

Now you have a third option: `browser-test.html`, the simplest way to
get automated tests running in the browser.

**Features:**

- Entire test framework lives in one html file. Just drop it into your
  project and write tests.
- Tests are run in the browser, so you have access to:
  - All browser APIs. Test against the DOM, IndexedDB, local storage, etc.
  - Dev tools. Set breakpoints, run things in the console
- Familiar testing syntax. Simple, but expressive.
- Bring your own browser means cross-browser testing is easy.
- Test framework is ~175 lines of easy to read javascript.

# Quickstart Usage

Using `browser-test.html` is as simple as **Copy, Serve, Write, Refresh**:

1. **Copy** the whole browser-test.html file into your project
2. **Serve** the browser-test.html file and your library on localhost
3. **Write** tests directly into the browser-test.html file
4. **Refresh** the browser page to see the results

## Copy

The browser-test.html file is MIT licensed, and has the attribution
included, so you can just copy it directly into your project. For
example, the following structure works:

```
your-project/
  readme.md
  src/
    index.js
  test/
    browser-test.html
```

## Serve

You'll need to bring your own file server and browser, but for the
example structure above, it could look as simple as:

```
cd your-project
caddy fileserver --location :8080
firefox http://localhost:8080/test/browser-test.html
```

## Write

Writing tests is as simple as adding directly to the `<script>` in
the `browser-test.html` file. You'll need to

```html
<script type="module">
  // import your code here
  import { yourFunction } from '/src/index.js'

  class Tester {
    // The browser-test framework is defined here
  }

  // Your tests go here
  describe("yourFunction", () => {
    expectEqual("Returns 1337", 1337, async () => {
      return await yourFunction();
    })
  })
</script>
```

*Note:* Your import path may change based on how you've structured your
project and how you're serving the files.

## Refresh

The results of your tests show up in the browser window. Green is
passing, Red is failing. To see new tests and results you just need to
refresh the window.

Because tests are running in your browser window, you also have full
access to debugging tools and the console during test runs.

*Note:* Depending on how your browser caches dependencies you may need
to do a hard refresh (e.g. Ctr-Shift-R on firefox) to pick up code
changes.

# API

The Testing API is a bit similar to Jest, but vaslty simplified.

- Use [`describe`](#describe) to create sections of tests
- Perform tests with the [`expect`](#the-expect-family) family of functions.
- Create fixtures with [`fix`](#fix) to pull out common setup and
  teardown functionality.

## `describe`

Create a new testing context. A new testing context means:

- A new section in the results output
- A new closure for definitions and fixtures

**Usage:** `describe(<name>, <body>)`

At it's simplest, `describe` can be used to structure your tests
hierarchicaly.

```javascript
describe("My Tests", () => {
  describe("Function 1", () => {
    expectEqual("Returns 100", 100, () => function1());
  })

  describe("Function 2", () => {
    expectEqual("Returns 200", 200, () => function2());
  })
});
```

More advanced uses are also possible. Within the body callback you can:

- Run arbitrary code that you can close over with tests or further
  `describe` calls.
- Run `fix` to attach fixtures to subsequent tests.
- Run any number of the `expect` family of functions to perform tests.
- Create arbitrarily nested contexts with more `describe` calls.
- Use `await` as long as the callback is marked as `async`.

An example that shows how these features could be used:

```javascript
describe('myClass', () => {
  const id = 123;
  const instance = myClass(id);

  expectEqual('id is set', id, () => {
    return myClass.id;
  })

  describe('myClass.myMethod', async () => {
    fix(
      'setup',
      async () => await instance.runSetup(),
      async () => await instance.runTeardown(),
    );

    expectEqual("Fetches correct default resource", 777, async ({ setup }) => {
      const resourceId = setup.id;
      const resource = await instance.myMethod(resourceId)
      return resource.value;
    });

    expectErr('invalid resource is an error', async () => {
      await instance.myMethod("INVALID")
    });
  })
})
```

## The `expect` family.

These functions represent the testing unit for the framework. There
are four different functions, useful for different scenarios:

- `expect`: Checks a result is `true`.
- `expectEqual`: Checks the result equals a value.
- `expectDeepEqual`: Checks a result deep into objects and arrays
- `expectErr`: Checks that an error is thrown

For all of the `expect` family of functions the first argument should
be the name of the test and the last argument is an (optionally async)
callback which runs your test code.

By default the callback you provide takes no arguments, and must
return the value to be compared against. In all of these functions
besides `expectErr` any unhandled error is treated as a test failure.

### `expect`

A test that passes when the provided callback returns true.

**Usage:** `expect(<name>, <callback>)`

**Examples:**

Simple tests:

```javascript
# Passes
expect("true", () => true);

# Fails:
expect("false", () => false);

# Fails:
expect("doesn't error", async () => {
  await Promise.reject("always throws");
  return true;
});
```

Sequences of tests. Since any error fails a test, `expect` can be used
with `assert`

```javascript
expect("Everything works", async () => {
  const data = await setup();
  assert(data !=== undefined, "Setup ran");

  const output = await process(data);
  assert(ouput === 4, "Output should be 4");

  const didCleanup = await cleanup(output);
  assert(didCleanup, "cleanup succeeded");

  return true;
})
```

### `expectEqual`

A test that passes when the provided callback returns a value equal to
the value provided.

**Usage:** `expect(<name>, <expected value>, <callback>)`

**Examples:**

```javascript
# Passes
expectEqual("1 + 1 = 2", 2, () => 1 + 1);

# Fails
expectEqual("2 + 2 = 5", 5, () => 2 + 2);

# Fails -- Must use expectDeepEqual
expectEqual("same array values", [1, 2, 3], () => [1, 2, 3]);
```

*Note:* Since object / array comparisons in javascript are done by
object identity, if you want to compare objects / arrays you must use
`expectDeepEqual` as described below

### `expectDeepEqual`

A test that passes when the provided callback returns a value

This is checked by encoding the provided value and the returned value
as JSON and doing a string comparison on the output.

**Usage:** `expectDeepEqual(<name>, <expected value>, <callback>)`

**Examples:**

```javascript
# Passes
expectDeepEqual("Arrays are equal", [1, 2, 3], () => [1, 2, 3])

# Passes
expectDeepEqual(
  "deep object comparison",
  {
    "first": [1, 2, 3],
    "second": {
      "third": {
        a: 1,
        b: 2,
        c: 3,
      },
    },
  },
  () => {
    return {
      "first": [1, 2, 3],
      "second": {
        "third": { a: 1, b: 2, c: 3, },
      },
    },
  }
)

# Fails
expectDeepEqual("Order matters", {a: 1, b: 2}, () => {b: 2, a: 1});
```

*Note:* Because the comparison is string comparison after JSON
encoding, object comparisons are (overly) sensitive to ordering. This
can be mitigated by either carefully arranging your field order in the
expected value or calling `Object.entries(val).toSorted()` on both
sides.

### `expectErr`

A test that passes when the provided callback encounters an unhandled
error.

**Usage:** `expectErr(<name>, <callback>)`

**Examples**:

```javascript
# Passes
expectErr("Throw an error", () => {
  throw new Error("Oops!");
})

# Passes
expectErr("Awaiting a rejected promise", async () => {
  await Promise.reject("Failed");
})

# Fails
expectErr("No error thrown", () => 3)
```

*Note:* `expectErr` does not detect the value of the error, so test
functions can pass for different errors than you intended. Be sure to
look at the test output in the browser window to ensure the error
you're seeing it the error your test expects.

## `fix`

Register a fixture to be run for every test in this text context and
descendent test contexts.

Before each tests is run, all fixtures in its context will be
run. Each test will be passed as it's only argument an object mapping
the name of each fixture to it's result. After each test is run, all
teardown functions will be run, with the fixture results passed in.

**Usage:** `fix(<name>, <setupFn>, [<teardownFn>])`

**Examples:**

```javascript
describe("Simple fixtures, no teardown required", () => {
  fix("three", () => 3);
  fix("asyncThree", async () => await 3)

  expectEqual("three is 3", 3, ({three}) => three);
  expectEqual("async three is 3", 3, ({asyncThree}) => three);
})

describe("Fixture with setup and teardown", () => {
  const data = {};
  fix(
    "dataStoreKey",
    () => {
      const uuid = crypto.randomUUID();
      data[uuid] = 12;
      return uuid;
    },
    ({dataStoreKey}) => {
      delete data[dataStoreKey];
    }
  );
  expectEqual("seeded data", 12, ({dataStoreKey}) => {
    return data[dataStoreKey]
  })
})

describe("Async fixtures", () => {
  fix("asyncThree", async () => await 3);
  fix("promiseThree", () => Promise.resolve(3));
  fix("closureThree", () => (() => Promise.resolve(3)));

  expectEqual("asyncThree", 3, ({asyncThree}) => asyncThree);
  expectEqual("promiseThree", 3, ({promiseThree}) => promiseThree);
  expectEqual(
    "closureThree", 3, async ({closureThree}) => await closureThree()
  );
})
```

*Notes:*

- All in-scope fixtures are run for every test, even if their results
are unused.

- Because tests are run asyncrounously, you cannot rely on a teardown
function running before another test's setup function has run. You
must still assume that any shared data can be updated in any order. In
general it is a simpler pattern to create new data for each test
function to own and then clean that up afterwards if necessary.

- If a fixture setup function errors, it will prevent tests that test
  context from running. This includes async fixtures that reject. If
  you need a promise from a fixture, return a closure instead.

## `assert`

Throw an error with a provided method if the provided value is not
`true`. Useful for checking intermediate steps in `expect` functions.

**Usage:** `assert(value, msg)`

**Examples:**

```javascript
# Passes
expect("several checks", () => {
  assert(1 + 1 === 2, "1 + 1 = 2");
  assert([1, 2, 3][1] === 2, "second element is 2");
  return true
})

# Fails
expect("Bad math", () => {
  assert(2 + 2 === 5, "2 + 2 = 5");
  return true;
})
```

## Built-in tests

The `browser-test.html` file comes with a set of example tests
built in that show usage of the various features. These can be used as
a reference and/or deleted from your project.

*Note:* These built-in tests also serve to test the testing
framework itself, so we actually expect some of them to be failing.

## Method Names

If you don't like the method names, renaming them is as simple as
changing the names in the destructuring from `test.methods()`. Just
find the lines that look like this:

```javascript
const {
  describe, expect, expectEqual, expectDeepEqual, expectErr
} = test.methods();
```

And give new names to the objects. For example:

```javscript
const {
  describe: desc,
  expect: test,
  expectEqual: testEq,
  expectDeepEqual: testDeep,
  expectErr: testErr,
} = test.methods();
```

# License

Browser-test is MIT licensed. See LICENSE.txt for the full license
text. When copying `browser-test.html` into your project, simply leave
the license notice at the top of the file intact.
