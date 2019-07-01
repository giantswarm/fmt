# JavaScript Style Guide

## Prettier

Our JavaScript Code Style is determined by `prettier`. Please make sure files have
passed through `prettier` before committing them to the repo. If you are using VS Code
and want to format your files you can write this into your preferences JSON:

```
{
  "files.associations": {
    "*.js": "javascriptreact"
  },
  "[javascript]": {
    "editor.formatOnSave": true
  }
}
```

A CI step will enforce that this has happened, failing the CI if it detects that
`prettier` would make any changes.

We use the following config params:
- `--jsx-single-quote`
- `--single-quote`
- `--trailing-comma es5`


## React Style Guide

## Props and State & ES6 Destructuring

Always destructure props and state to local variables if they are used more than
once inside the render method:

```
const AppDetailsBody = props => {
  const { title } = props;
  return  (
    <h1>{title}</h1>
    ...
    <p>This component has the following title: {title}</p>
  );
};
```

### Functional (or stateless) Components vs Class Components

For performance and readibility reasons we will use functional components when possible.
Class components are allright, of course, when we need to manage state.

<small>* Rewrite this when we decide how we are going to use Hooks</small>

### Handling events and  `this` binding

We will use arrow functions instead of binding in the constructor with the experimental
public class fields to correctly bind `this`:

```
class LoggingButton extends React.Component {
  // This syntax ensures `this` is bound within handleClick.
  // Warning: this is *experimental* syntax.
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```
<small>*Example from the React Official Documentation</small>

This is not the *standard* way of dealing with bindings but it is the most readable - and
popular - way of working. There are different ways of achieving the same result, each one
with its pros and cons. More info in the [React Docs](https://reactjs.org/docs/handling-events.html)

### Class Components Structure and ordering

- State
- Built-in methods
- Custom
- Render method
- Proptypes

### Naming

- Files: snake_case
- Props, state and methods: camelCase
- Components: PascalCase

## Parentheses

Wrap JSX tags in parentheses when they span more than one line, see [Airbnb Style Guide](https://github.com/airbnb/javascript/tree/master/react#parentheses) 

```
render() {
  return (
    <MyComponent variant="long body" foo="bar">
      <MyChild />
    </MyComponent>
  );
}
```

## CSS

Currently we are moving all the styles in sass files (.scss / .sass) to React components with
Emotion](https://emotion.sh). We have decided to use the css syntax instead of the JavaScript
object syntax.


## Redux

_Pending_

## Testing

We are going to use:

- [Jest](https://jestjs.io/) and [React Testing Library](https://testing-library.com/docs/react-testing-library/intro) for unit and integration tests
- [Cypress](https://www.cypress.io/) for E2E(End To End / functional) tests
- We may use [Browserstack](https://www.browserstack.com/) for crossbrowser testing