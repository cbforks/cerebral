# Test

Cerebral makes it easy to test your application components and business logic.

## Components
The **Container** you use to expose Cerebral to your components can also be used when testing.

```js
import React from 'react'
import {mount} from 'enzyme'
import {Container} from '@cerebral/react'

import Foo from './Foo'

describe('<Foo />', () => {
  it('allows us to set props', () => {
    const controller = Controller({
      // set test fixture state
      state: {
        foo: 'bar'
      },
      // set test fixture dummy signals needed by components
      signals: {
        rootSignalName: []
      }
      moduleName: {
        signals: {
          moduleSignalName: []
        }
      }
    })
    const wrapper = mount(
      <Container controller={controller}>
        <Foo />
      </Container>
    )
    expect(wrapper.find('.foo')).to.have.length(1)
  })
})
```

This approach allows you to pass down state wherever you want and ensure the components render as expected.

> If you want to test that signals are called, then you can add a mock action to the dummy signals and check that it was called

## Computes

The `runCompute` test helper accepts the `compute` and `fixture` arguments and returns the compute output.

```js
const result = runCompute(compute, fixture)
```

The optional `fixture` argument should be an object that contains any of the following:

```js
{
  state: {}, // test state
  props: {}  // props passed to the computed
}
```

### Example

```js
import {props, state} from 'cerebral/tags'
import {runCompute} from 'cerebral/test'

import multiply from './multiply'

it('should multiply by the specified number', () => {
  const result = runCompute(multiply, {
    state: { number: 5 },
    props: { multiplyBy: 2 }
  })
  assert.equal(result, 10)
})
```

## Actions

The `runAction` test helper accepts the `action` and `fixture` arguments and returns a promise.

```js
runAction(action, fixture).then((result) => {})
```

The optional `fixture` argument should be an object that contains any of the following:

```js
{
  state: {}, // test state
  props: {}, // props passed to the action
  // any other options that can be passed to the
  // cerebral controller, including router, providers...
}
```

The `result` object passed when the promise resolves contains `state`, `controller`, `props` and `output` properties.

```js
{
  state,
  controller,
  props: {
    // props data received by action
  },
  output: {
    // action output data
  }
}
```

### Example

```js
import {state} from 'cerebral/tags'
import {runAction} from 'cerebral/test'

import increment from './increment'

it('should increment numbers in state', () => {
  return runAction(increment, { state: { number: 1 } })
    .then(({state}) => assert.equal(state.number, 2))
})
```

## Signals

The `CerebralTest` factory returns runSignal, setState and getState functions as well as the controller.

```js
const cerebral = CerebralTest(fixture, options)
cerebral.setState(path, value)
cerebral.runSignal(signal, props).then((result) => {})
const value = cerebral.getState(path)
const wrapper = mount(
  <Container controller={cerebral.controller}>
    <Login />
  </Container>
)
```

The `fixture` argument will be passed to the cerebral controller so can contain the same properties (state, signals, modules, etc...). Note that state initialized in a module takes precedence over the state property of a fixture. Example:

```js
const fixture = {
  state: {
    app: {    
      showNavigation: true    
    }
  },
  modules: {
    app // if app initializes showNavigation as false, this will override the previous setting
  }
}
```

The optional `options` argument contain the the following options:

`recordActions: true|false|'byName'`

When `recordActions: true` is specified each action will record its props/output against its index within the signal action chain. When `recordActions: 'byName'` is specified each action will record its output against an named property in the result.

The `result` object passed when the promise resolves contains `state`, `controller` and an object for each named action in the signal chain with the same name as the actions with `props` and `output` properties.

### Example

```js
import {CerebralTest} from 'cerebral/test'

it('should accumulate a count', () => {
  const cerebral = CerebralTest({
    modules: {
      math: math()
    }
  })
  cerebral.setState('math.count', 0)
  return cerebral.runSignal('math.plusOne').then(({state}) => {
    assert.equal(state.math.count, 1)
    return cerebral.runSignal('math.plus', {value: 2}).then(() => {
      assert.equal(cerebral.getState('math.count'), 3)
    })
  })
})
```
