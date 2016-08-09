# Runnable Code and Test Patterns Guide

Runnable's approach to writing code and tests.

## Table of Contents

1. [Code Formatting](#code-formatting)
1. [Testing](#testing)
    1. [Types of Tests](#types-of-tests)
    1. [Stubbing](#stubbing)
    1. [Asserting](#asserting)
    1. [Additional Testing Reading](#additional-testing-reading)
1. [Configuration Files](#configs)

## Code Formatting

[![js-standard-style](https://cdn.rawgit.com/feross/standard/master/badge.svg)](https://github.com/feross/standard)

We use [`standard`](http://standardjs.com/) in most projects. If it passes `standard`, anything else is up to the developer. No questions asked.

**[back to the top](#table-of-contents)**

## Testing

Let's discuss various tests, what they should do and cover, and how best to organize them.

### Types of Tests

There are three types of tests we should be concerned about.

- Unit
- Integration
- Functional

#### Definitions

**Unit** tests are pretty obvious (or should be). They test exactly one unit of functionality. Most likely this should follow the boundaries of functions or at largest, modules. If we're finding we're writing a lot unit tests for a specific function, it may need to be broken up.

An important note: unit tests should _never_ touch an external service (e.g. database).

**Integration** tests make sure that one or more components function correctly together. This can be multiple functions within a module, multiple modules interacting with each other, or a module interacting with a database or external service.

**Functional** tests should align with product ideas. If a user goes through a given flow, they should finish with some outcome. We've found these tests to be pretty difficult to work with as of late, and are trying to get rid of some of these.

##### Where do you draw the line?

There will always be discussion on where the lines between (for example) **unit** and **integration** tests. For a unit test in a mongoose environment, mocking out the layer right at mongoose and simply returning sample objects is sufficient. The developer should have faith (and understanding) that mongoose will correctly return objects. If there's more paculiar logic that you may not be sure of or is more complex (like a map-reduce or aggrigation in mongo, or making sure populate is doing the correct thing), you would move to an **integration** test and write it there.

These definitely aren't hard-and-fast lines in the sand. A good goal would be that no external services are required to get unit tests off the ground and running, but integration tests may need more setup (like a database). When we find various tests that don't fit into the classification they're housed in, we can always fix the dependency or move the test when appropriate.

#### Organization

In most cases, we should attempt to organize our file structure as follows:

```
repository/
  lib/ (or something like it)
  tests/
    unit/
    integration/
    functional/
```

Additionally, the `package.json` should support the following commands:

- `npm test`: runs a code linter and all three sets of tests
- `npm unit`: runs the unit tests
- `npm integration`: runs the integration tests
- `npm functional`: runs the functional tests

### Stubbing

One of the most powerful tools we have (and are still learning to use well) is `sinon`. `sinon` is a stubbing and spying library that helps us assert that functions are called with what we expect.

`sinon`'s [docs](http://sinonjs.org/docs/) are quite good, but we will talk about some guidelines to follow when using it:

- In all tests where we are going to stub or spy on something, do so in a `before*`
- In all tests where we do stub or spy on something,
    - if it is on an object that is regenerated every test, no action is necessary
    - if it is on a module or something persistent, always restore the function in an `after*`
- Stub modules and functions in the correct scopes (meaning, as close to the test that uses it as possible, but hierarchical stubbing is fine)
- When stubbing functions, we may stub the function to a 'truthy' behavior; doing so reduces code duplication, though the developer will need to adjust the stub(s) to test error cases
- Collecting stubs into logical blocks and using the language to define what is being set up is encouraged (i.e. use `describe` blocks to collect and describe stubs, `it` blocks to say what happens)

### Asserting

Now that we have tests all squared away and written, let's talk about our assertions.

First, a few options we have been using are `code` and `chai.assert`. [`code`](https://github.com/hapijs/code) is pretty straight forward, using the `expect` form of assertions. [`chai`](https://github.com/chaijs/chai)'s assertion library has worked well, especially with [promises](https://github.com/domenic/chai-as-promised), and we have been using chai's `assert` which helps keep straight which library with which we are working.

We also have started using `sinon`'s assert features as well (e.g. `sinon.assert.calledWith`) which provides much nicer error messages than alternatives.

A few notes to help guide assertions:

- All tests must assert something
- When testing with a stub in the same logical group, we must assert something about the stub in at least one test. Should include
    - if it was called, or not
    - what it was called with, if applicable
- Stubs should (most likely) include checks from this list:
    - `sinon.assert.called`
    - `sinon.assert.notCalled`
    - `sinon.assert.calledOnce`, `sinon.assert.calledTwice`, etc.
    - `sinon.assert.callOrder`
    - `sinon.assert.calledWith`
    - `sinon.assert.calledWithExactly` (preferred over `calledWith`)
- When testing with promises (and `chai-as-promised`), always start with something such as `return assert.isFulfilled(myPromise())`, and always remember to return your promise.

### Additional Testing Reading

An interesting read on how 'value' could be perceived in testing is one of the posts on [Google's Testing Blog](http://googletesting.blogspot.com/2015/04/just-say-no-to-more-end-to-end-tests.html). It's not super long, so I encourage anyone to read. They very much encourage unit and integration tests over end-to-end (I read it as functional) tests as they help the developers much more and have the exact same value to the customer as any end-to-end test: a bug fix. If you can write a unit test for your bug rather than an end-to-end test, wouldn't you rather do that?

## Configuration

#### Naming

Repositories that need different configurations should have configuration files in the `config` directory. Files should be prefixed with `.env` and postfixed with the environment. Example `.env.production-gamma`. The `.env` file contains defaults shared across all environments.

#### Precedence

Here is the order of precedence with number 1 being the highest priority.

1. Environment variables
2. `.env.[environment]`
3. `.env`
 
#### Content

Configuration files should include everything needed to configures the application per environment with the exception of secret keys / passwords and connections. Only `.env.development` and `.env.test` can have these in them.

**[back to the top](#table-of-contents)**
