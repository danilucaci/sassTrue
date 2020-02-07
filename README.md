# Sass unit tests

Some say that if your code doesn't have tests, you don't have shippable code. I can agree with that, but when it comes to CSS or Sass unit testing there are few tools to choose from. But thankfully, there are tools and there is no excuse to not use them. 

This mini-tutorial will focus on setting up [Sass True](https://github.com/oddbird/true), the Sass unit testing framework developed by [Miriam Suzanne](https://github.com/mirisuzanne), a long time contributor to the Sass community and a true power user. 

### What is Sass unit testing?

Unit testing in Sass is not much different from unit testing in any other language. There are methods to define a test module, a method to wrap a series of tests, or assertions, then there are 4 assertion-types: `assert-true`, `assert-false`, `assert-equal` and `assert-unequal`.

The only other thing to consider is that there are two different patterns to follow for testing. One for functions that will evaluate the output of the function, the other is for mixins which expects a specific return based on the configuration of the mixin. 

### Install Sass True

For the sake of clarity, this tutorial is going to assume that there are no other files than the Sass we want to test and the test files. 

With a clean directory, you want to run the following install. 

```
$ npm i node-sass sass-true glob jest
```

Now create two directories 

```
$ mkdir src tests
```

In your `package.json` update to the following

```json
"scripts": {
  "test": "jest"
},
```

With everything installed, let's write some code. 

#### Get Sass True to talk to Jest

This is the last part of the setup needed to get the `jest` command working with Sass True. We need to write a small js shim. 

In the `tests` dir, create the new shim file

```
$ touch tests/scss.spec.js
```

In this new file, add the following code

```js
const path = require('path')
const sassTrue = require('sass-true')
const glob = require('glob')

describe('Sass', () => {
  // Find all of the Sass files that end in `*.spec.scss` in any directory in this project.
  // I use path.resolve because True requires absolute paths to compile test files.
  const sassTestFiles = glob.sync(path.resolve(process.cwd(), 'tests/**/*.spec.scss'))

  // Run True on every file found with the describe and it methods provided
  sassTestFiles.forEach(file =>
    sassTrue.runSass({ file }, { describe, it })
  )
})
```

This shim is taking the instructions from the Sass True docs and adding some superpowers to make things much easier. Mainly, this is automatically looping through all the spec files so that you don't have to write out all the absolute paths to the files as Sass True requires. 

With this in place, we are ready to start writing tests. 

### The Sass function

Sass True only will test mixins and functions. After all, that is all you need to ensure that your tooling is always working correctly. Testing the CSS output itself is another tool's job. 

In your project create the new Sass file that we will use as the function to test.

```
$ touch src/_map-deep-get.scss
```

In the file, add the following code.

```scss
/// This function is to be used to return nested key values within a nested map
/// @group utility
/// @parameter {Variable} $map [null] - pass in map to be evaluated
/// @parameter {Variable} $keys [null] - pass in keys to be evaluated
/// @link https://css-tricks.com/snippets/sass/deep-getset-maps/ Article from CSS-Tricks
/// @example scss - pass map and strings/variables into function
///   $map: (
///     'size': (
///       'sml': 10px
///     )
///   );
///   $var: auro_map-deep-get($tokens, 'size', 'sml'); => 10px
///

@function auro_map-deep-get($map, $keys...) {
  @each $key in $keys {
    $map: map-get($map, $key);
  }
  @return $map;
}
```

This is a pretty common function that is added to Sass projects. If you are not familiar with this technique, please be sure to read the [CSS-Tricks](https://css-tricks.com/snippets/sass/deep-getset-maps/) article. 

### Testing a Sass function

Let's create the new file we need to test the Sass. Remember in the shim we are looking for any file that matches `*.spec.scss`, so to test our map function, we can create a file name like this. 

```
$ touch tests/mapDeep.spec.scss
```

What's powerful with Sass True is that it's written with Sass. Because of this, all the feature of Sass are available to you when writing tests. 

In this new file, let's import the dependencies we need. First, we need to import True, then we need to import the Sass function we want to test.  

```scss
@import 'true';
@import '../src/map-deep-get';
```

Looking at the code for the map function, it's clear that we need a map to use in this test.

```scss
$map: (
  'size': (
    'sml': 10px
  )
);
```

For the test, the first thing needed is to describe the test you are running. This will help you see the results of the test in the CLI output. 

To me, the docs were a little hard to follow as they still describe the actual mixins used in Sass True, it appears for compatibility with testing frameworks like Jest, several aliases were created. I will try my best to address these cross references in the code. 

Remember in the shim we have the argument of `{ describe, it }`. In Sass True, for example, `@mixin test-module()` is aliased as `describe()` and the `test()` mixin is aliased to the `it()` mixin. 

```scss
@include describe('map-deep-get()') {
  ...
}
```

Next, you need to define the test. This will use the `it()` mixin.

```scss
@include describe('map-deep-get()') {
  @include it('should return the value from a deep-map request') {
    ...
  }
}
```

Last we need the test itself. In this example, we will use the `asset-equal` method. 

> Assert that two parameters are equal Assertions are used inside the `test()` mixin to define the expected results of the test.

The `assert-equal()` mixin takes 4 arguments, but we are only going to use `$assert` and `$expected`. 

```scss
assert-equal($assert, $expected);
```

Inside the parens of the `assert-equal()` mixin put the function as if we were to use if in Sass. As illustrated, we can also take advantage of the `$map` variable we set earlier. 

__ProTip:__ in your project, if you have a series of variables you are already using and need these available for the test, simply `@import` them before running the test(s). 

For the `$expected` argument of the test, we put the expected value. 

```scss
@include describe('map-deep-get()') {
  @include it('should return the value from a deep-map request') {
    @include assert-equal(
      map-deep-get($map, 'size', 'sml'), 10px
    );
  }
}
```

At this point, you should have a setup that has all the libraries needed, a starter function to test and a working test. 

### Running the test

At this point, it's all downhill. From the command line, run your test. 

```
$ npm test
```

All things working correctly, you should see the following:

```
PASS  tests/scss.spec.js
  Sass
    map-deep-get()
      ✓ should return the value from a deep-map request (1ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        1.422s
Ran all test suites.
```

If at this point you do not have success, please review the tutorial so far and/or compare your code to [this demo repo](https://github.com/blackfalcon/sassTrue).


### The Sass mixin

At this point, you should have Sass True set up and have the map function test working. Moving forward you may want to test a mixin. Testing a mixin is only slightly different than testing a function. For the mixin test, let's test a common pattern of creating a breakpoint mixin. 

```
$ touch src/_breakpoint--lg.scss
```

The mixin itself will look like the following: 

```scss
/// Standard breakpoint to support resolutions greater than 1232px.
/// @group responsive
/// @example scss - Set breakpoint
///   .breakpoint--lg {
///       @include breakpoint--lg {
///         color: orange;
///       }
///   }
@mixin breakpoint--lg {
  @media screen and (min-width: $breakpoint-lg) {
    @content;
  }
}
```

### Testing the Sass mixin

In the mixin, you may have noticed that there is the global var `$breakpoint-lg` as part of the mixin. To make this a little more 'real-world', let's create a global vars file and define the value of this variable. 

```
$ touch src/_globals.scss
```

And in this file will put the variable and its value. 

```scss
$breakpoint-lg: 1232px;
```

Getting to the test, let's create the test file. 

```
$ touch tests/breakpoint.spec.scss
```

And to start this file, just like previously, we will define all our dependencies. 

```scss
@import 'true';
@import '../src/globals';
@import '../src/breakpoint--lg';
```

Next, just like the function, we need to describe the test. 

```scss
@include describe('breakpoint--lg()') {
  ...
}
```

Next, we again define what `it` is expected to do. 

```scss
@include describe('breakpoint--lg()') {
  @include it('should return content within pre-defined media query') {
    ...
  }
}
```

With the mixin we can simply use the `assert()` method for our test. 

```scss
@include describe('breakpoint--lg()') {
  @include it('should return content within pre-defined media query') {
    @include assert {
        ...
    }
  }
}
```

Within the `assert()` method we want to test the content to be evaluated using the `output()` method. 

> Describe the test content to be evaluated against the paired `expect()` block. Assertions are used inside the `test()` [or `it()`] mixin to define the expected results of the test.

Ok, so that simply means that within the `output()` mixin you can include whatever Sass you want to use that uses the mixin you want to test. For example, let's create a selector that uses the breakpoint mixin we want to test and inject the `@content` value inside the mixin. 

```scss
@include describe('breakpoint--lg()') {
  @include it('should return content within pre-defined media query') {
    @include assert {
      @include output {
        .breakpoint--lg {
          @include breakpoint--lg {
            color: orange;
          }
        }
      }
    }
  }
}
```

For the last part, we need an assertion of what the output CSS will be. For this, we will use the `expect()` method. 

> Describe the expected results of the paired `output()` block. The `expect()` mixin requires a content block and should be nested inside the `assert()` mixin, along with a single `output()` block. Assertions are used inside the `test()` mixin to define the expected results of the test.

So within the `expect()` mixin we just add in the expected CSS output. 

```scss
@include describe('breakpoint--lg()') {
  @include it('should return content within pre-defined media query') {
    @include assert {
      @include output {
        .breakpoint--lg {
          @include breakpoint--lg {
            color: orange;
          }
        }
      }

      @include expect {
        @media screen and (min-width: 1232px) {
          .breakpoint--lg {
            color: orange;
          }
        }
      }
    }
  }
}
```

### Running the test

At this point, you have the `Sass` test suite with two individual tests running one assertion each. 

Running the `$ npm test` command, you should see the following: 

```
PASS  tests/scss.spec.js
  Sass
    breakpoint--lg()
      ✓ should return content within pre-defined media query (2ms)
    map-deep-get()
      ✓ should return the value from a deep-map request

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        1.512s
Ran all test suites.
```

### Conclusion

Again, if at this point you do not have success, please review the tutorial so far and/or compare your code to [this demo repo](https://github.com/blackfalcon/sassTrue).

Today's code is getting more and more complex, not having a way to ensure that as new code is being added that all your assumptions are still true is really risky. 

I hope you find this mini-tutorial helpful and happy testing! 
