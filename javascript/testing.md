# Frontend Testing Manifesto

### Statements

- Testing UI [is hard]
- And it's not easy to [know what to test](https://twitter.com/dan_abramov/status/1197901310982590464)
- Mocking too much reduces test quality
- Test from a user perspective. Think less about the code you are testing and more about the use cases that code supports

### Tools

Tools we are using:

- [Jest](https://jestjs.io/en/): assertions, matchers and mocking
- [Jest DOM](https://github.com/testing-library/jest-dom#table-of-contents): custom matchers that extend Jest
- [DOM Testing Library](https://testing-library.com/docs/dom-testing-library/cheatsheet): basically for querying the DOM
- [React Testing Library](https://testing-library.com/docs/react-testing-library/cheatsheet): for testing React components. Built on top of DOM Testing Library

### Snapshots

Use them just in components that we are not changing too frequently. Otherwise we will have to update them every now and then.

### Avoid implementation details

We want our tests to be [as resilient as possible](https://twitter.com/dan_abramov/status/1151842775417151489). Hence, don’t test if a css class is present or a piece of state holds this or that value, test what is actually rendered. For example:

- Test if a button is red instead of it has an `alert` class
- Test if an element is present in the DOM instead of testing a state value in a component

### Finding elements in the DOM
- You should look for elements in [the same way the user would look for an element](https://testing-library.com/docs/guide-which-query)

### Unit tests vs Integration tests

Integration tests give more test coverage with fewer tests while unit tests are a nice way of testing complex parts of our codebase.

Let's [keep a balance](https://twitter.com/rauchg/status/807626710350839808?lang=en) between them and keep in mind that more isolation = less bugs caught.

### What we think makes a good test

It:

- Runs fast
- Doesn't break often
- Easy to read/understand
- Catches bugs
- Gives good coverage to effort ratio


### Persisted nocks considered a antipattern

We should avoid using persisted nocks so that we get a good idea of the request
behaviour of our components. You might run into trouble when trying to do this
especially when combining it with `nock.disableNetConnect();` and `nock.cleanAll();`
in your `afterEach` block.

If you see errors after your test has finished running like:

```
Error: Nock: No match for request
```

or

```
NetConnectNotAllowedError
```

What is really happening is that the test has already finished but an asynchronous
part of the component is still running.

Possibly some request chain hasn't finished yet and it is trying to complete the chain.

However, since the test is considered done, the code has already moved on and
cleaned up all the nock interceptors, resulting in errors, sometimes non-deterministically
since it depends on when afterEach is called and when the asynchronous part of
the component in test gets time to run.

In these cases you should wait for all pending mocks to finish as part of your
test:

```
await wait(() => {
  expect(nock.isDone()).toBe(true);
});
```

That way your test will only be considered finished when all requests you thought
it would do are finished. It will also reveal any unecessary nocks, since after
5 seconds of nock not being done, this block will fail, and the whole test will
fail as well.