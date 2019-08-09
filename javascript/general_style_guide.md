# Frontend Style Guide

## Javascript

### `const`, `let` and `var`

Use `const`and `let`instead of `var`. Read [in this post](https://mathiasbynens.be/notes/es6-const) the benefits of this approach.

### Triple equal

Please use always triple equal comparisions and get rid of a lot of bugs related to types. With `===` we are comparing values **and types**, otherwise we are just comparing values:

```javascript
"0" == 0 // true
"0" === 0 // false
```

## Prettier

Our JavaScript Code Style is determined by `prettier`. Please make sure files have
passed through `prettier` before committing them to the repo. If you are using VS Code
and want to format your files you can write this into your preferences JSON:

```javascript
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

### Props and State & ES6 Destructuring

Always destructure props and state to local variables if they are used more than
once inside the render method:

```javascript
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

```javascript
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
- Custom methods
- Render method
- Proptypes

### Naming

- Files: snake_case
- Props, state and methods: camelCase
- Components: PascalCase

### Parentheses

Wrap JSX tags in parentheses when they span more than one line, see
[Airbnb Style Guide](https://github.com/airbnb/javascript/tree/master/react#parentheses) 

```javascript
render() {
  return (
    <MyComponent variant="long body" foo="bar">
      <MyChild />
    </MyComponent>
  );
}
```

### Importing images

Avoid `src` `img` attribute for local images. Instead, import them in React components as regular
ES6 modules with the `import` syntax and let Webpack handle them:

```javascript
import { logo } from '../images/;
```

## CSS

Currently we are in the process of moving all the styles in sass files (.scss / .sass) to React
components with Emotion](https://emotion.sh). We have decided to use the css syntax instead of
the JavaScript object syntax.

### Positioning

For positioning elements please don't use margins/paddings in those elements inside a container,
instead use **properties in the parent element** like `padding`, or in case of flexbox layouts,
`justify-content` or `align-*`. For instance, instead of this:

```css
.wrapper {
    background: red;
}

h3 {
    color: red;
    margin-top: 20px
}
```

Do this:

``` css
.wrapper {
    background: red;
    padding-top: 20px;
}

h3 {
    color: red;
}
```

This makes it easier to reuse element tags inside a wrapper element or component without inheriting 
positioning properties. 

However, we can use `margin`, `padding` or even `algin-*` to **make room between two sibling elements**.


## Redux

_Pending_

## Testing

We are going to use:

- [Jest](https://jestjs.io/) and [React Testing Library](https://testing-library.com/docs/react-testing-library/intro) 
for unit and integration tests
- [Cypress](https://www.cypress.io/) for E2E (End To End / functional) tests
- We may use [Browserstack](https://www.browserstack.com/) for crossbrowser testing
