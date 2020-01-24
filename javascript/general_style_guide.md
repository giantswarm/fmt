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

### Strings

Use single quotes for strings: `'This is a regular string` 

Use template literals (template strings) for interpolation or multiline strings:

```javascript
const name = 'John Doe';

// This is a template literal as we are using backticks here
const wave = `Hi ${ name }`;
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

- Regular JS files (not components) and non-component folders: camelCase.js
- Component files and folders: PascalCase
- Props, state and methods: camelCase
- We don't use `index.js` for component files.

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
components with [Emotion](https://emotion.sh). We have decided to use the css syntax instead of
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

### Libraries we use

- We use [Thunk](https://github.com/reduxjs/redux-thunk) to write async logic that interacts with the store
- We use [Immer](https://github.com/immerjs/immer) for creating the next state in Redux store in an immutable way
- We derive state in selectors using [Reselect](https://github.com/reduxjs/reselect)

### Our conventions

- Reducers are pure functions, we want reducers to be as clean as possible to easily see at a glance what pieces of state we are creating/deleting/updating
- We inline objects inside `dispatch()` instead of writing action creators
- We have a `loadingReducer` that takes care of all our loading flags

### Data flow

```

                         ╔════════════╗
                         ║   A P I    ║
                         ╚════════════╝
                               ↑
                               ↓
╔════════════╗           ╔════════════╗           ╔════════════╗
║  Batched   ║ → → → → → ║   Thunks   ║ → → → → → ║  Reducers  ║
║  Actions   ║           ╚════════════╝           ╚════════════╝
╚════════════╝                                          ↓
     ↑                                                  ↓
     ↑                                                  ↓
╔════════════╗           ╔════════════╗           ╔════════════╗
║ Components ║ ← ← ← ← ← ║  Selectors ║ ← ← ← ← ← ║   Store    ║
╚════════════╝           ╚════════════╝           ╚════════════╝
```

__Batched actions__: what we call a _batched actions function_ is a function where we call a series of _thunks_ that will result in a bunch of data that a specific view needs. We use async logic in those functions so we can wait for some API calls to be fulfilled (and its returned data saved in the store) before proceeding with other calls that might need this data.

These functions not only trigger these calls, but also aim to describe what exactly is going on in each view regarding the data layer.

__Thunks__ live in `actions/xxxxActions.js` files. These functions are responsible of

- Making API calls
- Transform the data received if it is needed so it fits in Redux store
- Dispatch actions

__Reducers__ modify state in the __store__.

__Selectors__ (when used) select or get data from the store.

__Components__ use this data to render the views.

### Loading flags

`loadingFlags` is an object at the root of our store where we save our loading flags for each action. `loadingReducer` reducer will take care of all our flags without us having to even think about them. It will update flags in store in the following manner: __any action dispatched__ that has a type with the `_REQUEST` suffix will update (or create if it doesn't exist) a flag in the store for its respective action. For example, if we dispatch an action with the type of `CLUSTERS_LOAD_REQUEST`, it will result in the following changes in the store:

```javascript
...
loadingFlags: {
  ...
  CLUSTERS_LOAD: true
}
```

On the contrary, if we dispatch any action with any of these suffixes: `_SUCCESS`, `_ERROR`, `_FINISHED`, `_NOT_FOUND`, the respective flag will be set to false. Hence, if we dispatch an action with the type of `CLUSTERS_LOAD_SUCCESS`, it will result in the following changes in the store:

```javascript
...
loadingFlags: {
  ...
  CLUSTERS_LOAD: false
}
```

#### Using loading flags

Normally we are going to dispatch actions in _thunks_. We can exceptionally dispatch them in other places, as we do in `Organizations/Detail.js` to address a very specific issue that happens when accessing a cluster directly (using a URL). But we should aim to dispatch any action in _thunks_ - or action creators - in `xxxxxActions.js` files as this is the natural place for actions to be dispatched and also for easy debugging.

This way, when you want to inspect which actions are dispatched when a `batchedXxxxx()` function is called, you just have to take a look at each one of the _thunks_ called in this batched function.

_Side note: not all actions are using the `_REQUEST` suffix, by now just some actions have been changed. (This and removing all loading flags outisde `store.loadingFlags` are TODOs in Oriol's personal board)_

#### Using loading flags in components

Thanks to these flags we have fine-grained control of what is going on in the store. We are able to know when a specific piece of data is available in the store. This way, we can use loading flags to:

- Prevent rendering views before state is updated
- Prevent undesired behaviors such as flashing in components
- Tell Happa exactly what views it can render in which moments

We can now have a parent view for example that waits for a specific flag to be set to `false` (most of the times an API call) and a child view that waits for another flag, so the we can improve the time to first render. For example:

```javascript
class Layout extends React.Component {
  ...
  render(){
    return(
      // This won't render anything but the spinner until this flag is set to false
      <LoadingOverlay loading={this.props.loadingClustersList}>
      // nested components
      <LoadingOverlay/>
    )
  }
}
...

function mapStateToProps(state) {
  return {
    // This is th flag we are using
    loadingClustersList: state.loadingFlags.CLUSTERS_LIST,
  };
}

```

This is a granchild component of `Layout`:

```javascript
// ClusterDashboardResources.js
function ClusterDashboardResources(){
...
  // We don't want to render this component until these flags are set to false 
  const loading =
    loadingClusters || loadingStatus || (isNodePool && loadingNodePools);

  return (
    <ClusterDetailsDiv>
      <LoadingOverlayWithoutStyles loading={loading}>
      // resources data
    <ClusterDetailsDiv>
  )
}
...

const mapStateToProps = (state, props) => {
  return {
    // These are exactly the flags we are using
    loadingClusters: state.loadingFlags.CLUSTERS_LOAD_DETAILS,
    loadingNodePools: state.loadingFlags.NODEPOOLS_LOAD,
    loadingStatus: state.loadingFlags.CLUSTER_LOAD_STATUS,
  };
};
```

#### Conventions in options object in batched functions

We are using a last argument in batched functions which is an object with the following booleans:

##### `filterBySelectedOrganization`

We use this to filter the clusters before we fetch details for them. If set to true Happa will fetch details and node pools for clusters owned by the selected organization.

##### `withLoadingFlags`

In cases such as batched functions called for __refreshing data which is already in store__ - ie, `batchedRefreshClusters()` or `batchedRefreshClusterDetailView()` - we don't want to wait for any flag to be set to `false`, so we are passing this parameter set to `false` to tell the _thunk_ to skip the dispatching of the loading flag, and so no spinner or placeholder will be rendered.

##### `initializeNodePools`

This option will reset `cluster.nodePools` to an empty array in the store. This is the array that contain node pools ids for a given cluster. We just do this the first time we fetch details for a cluster.


## Testing

We are using:

- [Jest](https://jestjs.io/) and [React Testing Library](https://testing-library.com/docs/react-testing-library/intro) for unit and integration tests

We are considering using:

- [Cypress](https://www.cypress.io/) for E2E (End To End / functional) tests
- We may use [Browserstack](https://www.browserstack.com/) for crossbrowser testing
