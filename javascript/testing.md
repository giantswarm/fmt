# Frontend Testing Examples

Tools we are using:

- [Jest](https://jestjs.io/en/): assertions, matchers and mocking
- [`jest-dom`](https://github.com/testing-library/jest-dom#table-of-contents): custom matchers that extend Jest,
- [DOM Testing Library](https://testing-library.com/docs/dom-testing-library/cheatsheet) and [React Testing Library](https://testing-library.com/docs/react-testing-library/cheatsheet): basically querying. Built on top of DOM Testing Library

### Test _it renders without crashing_

```javascript
test('renders without crashing', () => {
  const {container} = render(<ClusterApps someProp="some value" />)
})
```

### Snapshots

[Snapshots](https://jestjs.io/docs/en/snapshot-testing#snapshot-testing-with-jest) are 
a great way to test components that you __don't want to be changed__.

```javascript
test('matches snapshot', () => {
  const {container} = render(<ClusterApps someProp="some value" />)
  expect(container.firstChild).toMatchSnapshot()
})
```

### Assert rendered element (an error here) is triggered with an interaction

```javascript
test('entering an invalid value shows an error message', () => {
  const {getByLabelText, getByTestId} = render(<ViewAndEdit />)
  const input = getByLabelText(/name/i)
  fireEvent.change(input, {target: {value: 'a'}})
  expect(getByTestId('error-message')).toHaveTextContent(
    /please use a name with at least 3 characters/i,
  )
})
```

### Mock an API call after clicking a button

This is an imaginary component that should load to an element with `test-id="cluster-name"`
what's returned from the API call triggered when clicking the button

```javascript
// ...
import {loadClusterName as mockLoadClusterName} from '../api'
// ...

jest.mock('../api', () => {
  return {
    loadClusterName: jest.fn(subject =>
      Promise.resolve({data: {name: `Awesome cluster`}}),
    ),
  }
})

test('loads cluster name on click', async () => {
  const {getByText, getByTestId} = render(<ClusterNameLoader />)
  const loadButton = getByText(/load/i)
  fireEvent.click(loadButton)
  expect(mockLoadClusterName).toHaveBeenCalledTimes(1)
  await wait(() => expect(getByTestId('cluster-name')).toHaveTextContent('Awesome Cluster'))
})
```

### Test a redux connected React Component

```javascript
import {createStore} from 'redux'
import {Provider} from 'react-redux'
import {render, fireEvent} from 'react-testing-library'
import {clusterReducer, Cluster} from 'index'

test('can render with redux with custom initial state', () => {
  const store = createStore(reducer, {/* initial state here */})
  const {getByTestId, getByText} = render(
    <Provider store={store}>
      <Cluster />
    </Provider>,
  )
  // expect something here
})
```
