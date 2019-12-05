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
- Test if an element is in present in the DOM instead of testing a state value in a component

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
