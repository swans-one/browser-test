# Browser Test

Some javascript requires a browser context to run making them hard to
test. This simple test framework is an easy way to automate tests in
the browser without having to setup complex mocks or browser
automation tools.

**Features:**

- Entire test framework fits in one html file
- Hierarchical test organization: `it`.
- Simple but expressive testing functions:
  - `expect`: Checks a result is `true`.
  - `expectEqual`: Checks the result equals a value.
  - `expectDeepEqual`: Checks a result deep into objects and arrays
  - `expectErr`: Checks that an error is thrown
- Provide fixtures (coming soon)
- Tests run in your browser, so you have full access to dev tools for
  debugging.

# Quickstart Usage

Using `browser-test.html` is as simple as Copy, Serve, Write, Refresh:

1. Copy the whole browser-test.html file into your project
2. Serve the browser-test.html file and your library on localhost
3. Write tests directly into the browser-test.html file
4. Refresh the browser page to see the results

**Copy**

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

**Serve**

You'll need to bring your own file server and browser, but for the
example structure above, it could look as simple as:

```
cd your-project
caddy fileserver --location :8080
firefox http://localhost:8080/test/browser-test.html
```

**Write**

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
  it("yourFunction", () => {
    expectEqual("Returns 1337", 1337, async () => {
      return await yourFunction();
    })
  })
</script>
```

*Note:* Your import path may change based on how you've structured your
project and how you're serving the files.

**Refresh**

The results of your tests show up in the browser window. Green is
passing, Red is failing. To see new tests and results you just need to
refresh the window.

Because tests are running in your browser window, you also have full
access to debugging tools and the console during test runs.

*Note:* Depending on how your browser caches dependencies you may need
to do a hard refresh (e.g. Ctr-Shift-R on firefox) to pick up code
changes.

# API

# License
