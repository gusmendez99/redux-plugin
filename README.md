# Redux Plugin
[Redux](http://redux.js.org/) sub-architecture for 'plugin' [React](https://facebook.github.io/react/) components.

## Installation
```
npm install redux-plugin --save
```

## Motivation
Several times you will be writing reducers that manages multiple homogeneous state trees. For example, I wrote a PopOver component whose state is managed by redux. This component and it's reducer handles the basic tasks:
- Open / Close specific popover
- Open / Close all popovers
- Open / Close specific group of popovers
- Render another component as the popover content

Each instance of the PopOver component has it's own redux state sub-tree, coordinated by the `popover` reducer. The popover reducer use reducer composition to handle each popover individual state.

`redux-plugin` helps you defining plugin components and composed plugin reducers.

## Key concepts
`redux-plugin` supposes a special mindset about components: the *plugin components*. I encourage reading this section before jumping to the library, in order to see if this specific mindset fits your project.

### Plugin reducer
The plugin reducer manages multiple instances of each plugin element state. For example, the `popover` plugin reducer manages the following part of the state:

```Javascript
{
  ...,
  popover: {
    navNotifications: {
      isOpen: false,
      groups: ['popovers', 'tour-1'],
      configuration: {
        vertical: "below",
        horizontal: "center",
        withTip: true
      }
    },
    submitTooltip: {
      isOpen: true,
      groups: ['tooltips', 'tour-2'],
      configuration: {
        vertical: "above",
        horizontal: "center",
        withTip: true
      }
    },
    formError: {
      isOpen: false,
      groups: ['errors'],
      configuration: {
        vertical: "above",
        horizontal: "right",
        offset: 25,
        withTip: true
      }
    }
  }
}
```
The creation of a plugin reducer happens with a higher order reducer provided by `redux-plugin`:

```Javascript

import { pluginReducer as plugin } from 'redux-plugin';

const popover = plugin({
  name: 'popover',
  prefix: '@@redux-popover',
  reducer: popoverElement
});

```

If the plugin reducer receives a global action (ie. paylod has not the `id` key), it iterates and passes down the action to all the plugin element reducers.

### Plugin element reducer
It's a plain old reducer that manages the state for each plugin element (you just have to think of the state transitions of one popover!).

```Javascript
import { combineReducers } from 'redux';

const popoverElement = combineReducers({
  isOpen,
  groups,
  configuration
});
```

### Plugin element
Each instance of a plugin. It's state is managed by the element reducer, but proxied by the plugin reducer.

### Plugin element component
Dummy (presentational) component that takes simple props and renders the a plugin element instance (you just have to think the rendering of one popover!).

### Plugin component
Smart (connected) component that takes an `id` as prop, fetches the specific plugin element state and passes it down to the plugin element component. This component is generated by a higher order component.

```Javascript
import { pluginComponent as plugin } from 'redux-plugin';
import { register } from './actions';
import Popover from './components/popover';

export const ReduxPopover = plugin({
  defaultStateKey: 'popover',
  registerPluginElement: register
})(Popover);

```

### Registration action creator
Simple action creator that handles the registration of a new instance of a plugin (plugin element). The signature of this action creator should be `(id, initialState)`, where id is the plugin element id, and initialState is the first state of this plugin element.

```Javascript
export const register = (id, { isOpen = false, groups = [], configuration }) => ({
  type: types.POPOVER_REGISTERED,
  payload: { id, isOpen, groups, configuration }
});
```

### Plugin
A plugin is the combination of a plugin reducer and a plugin component.

## Use
Read the descriptions above to understand how to create the Popover component, then the following code should make sense:

```Javascript
import { Popover } from './components/popover';

const PopoverContent = () => (
  <div>
    <h1>Hi!</h1>
    <p>This is the popover content!</p>
  </div>
);

const View = () => (
  <Popover
    id='buttonPopover'
    initialState={ { isOpen: true } }
    content={ PopoverContent }>
    <button>Open!</button>
  </Popover>
);
```

## Contributors

[@samuelchvez](https://github.com/samuelchvez)

## License

Copyright (c) 2017 Samuel Chávez

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
