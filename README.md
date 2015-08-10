# `redux-immutable`

[![Travis build status](http://img.shields.io/travis/gajus/redux-immutable/master.svg?style=flat-square)](https://travis-ci.org/gajus/redux-immutable)
[![NPM version](http://img.shields.io/npm/v/redux-immutable.svg?style=flat-square)](https://www.npmjs.org/package/redux-immutable)

This package provides a single function `combineReducers`, which enables:

* Immutable state of the app.
* [Canonical Reducer Composition](https://github.com/gajus/canonical-reducer-composition).

`redux-immutable` `combineReducers` is inspired by the Redux [`combineReducers`](http://gaearon.github.io/redux/docs/api/combineReducers.html) and [Redux Reducer Composition](#redux-reducer-composition) pattern.

## Initial State

You must provide Redux `createStore` with the initial state using Immutable data.

```js
import {
    createStore
} from 'redux';

import {
    combineReducers
} from 'redux-immutable';

import Immutable from 'immutable';

import * as reducers from './reducers';

let app,
    store,
    state = {};

state.ui = {
    activeLocationId: 1
};

state.locations = [
    {
        id: 1,
        name: 'Foo',
        address: 'Foo st.',
        country: 'uk'
    },
    {
        id: 2,
        name: 'Bar',
        address: 'Bar st.',
        country: 'uk'
    }
];

state = Immutable.fromJS(state);

app = combineReducers(reducers);
store = createStore(app, state);

export default store;
```

## Unpacking Immutable State

`redux-immutable` `combineReducers` turns state into Immutable data. Therefore, when you request the store state you will get an instance of `Immutable.Map`:

```js
let state;

state = store.getState();

console.log(Immutable.Map.isMap(state));
// true
```

You can convert the entire state to a raw JavaScript object using Immutable.js [`toJS()`](https://facebook.github.io/immutable-js/docs/#/Iterable/toJS) function:

```js
state = store.getState().toJS();
```

The disadvantage of this method is that it will create a new JavaScript object every time it is called:

```js
console.log(state.toJS() === state.toJS());
// false
```

Because shallow comparison says that new state is different from the previous state you cannot take advantage of [PureRenderMixin](https://facebook.github.io/react/docs/pure-render-mixin.html) or an equivalent logic that manages [`shouldComponentUpdate`](https://facebook.github.io/react/docs/component-specs.html#updating-shouldcomponentupdate) using shallow object comparison.

For the above reason, you should refrain from converting state or parts of the state to raw JavaScript object.

## Using with [webpack](https://github.com/webpack/webpack) and [Babel](https://github.com/babel/babel)

The files in `./src/` are written using ES6 features. Therefore, you need to use a source-to-source compiler when loading the files. If you are using webpack to build your project and Babel, make an exception for the `redux-immutable` source, e.g.

```js
var webpack = require('webpack');

module.exports = {
    module: {
        loaders: [
            {
                test: /\.js$/,
                exclude: [
                    /node_modules(?!\/redux\-immutable)/
                ],
                loader: 'babel'
            }
        ]
    }
};
```

## Example

### `store.js`

```js
import {
    createStore
} from 'redux';

import {
    combineReducers
} from 'redux-immutable';

import Immutable from 'immutable';

import * as reducers from './reducers';

let app,
    store,
    state = {};

state.ui = {
    activeLocationId: 1
};

state.locations = [
    {
        id: 1,
        name: 'Foo',
        address: 'Foo st.',
        country: 'uk'
    },
    {
        id: 2,
        name: 'Bar',
        address: 'Bar st.',
        country: 'uk'
    }
];

state = Immutable.fromJS(state);

app = combineReducers(reducers);
store = createStore(app, state);

export default store;
```

### `reducers.js`

```js
/**
 * @param {Immutable} state
 * @param {Object} action
 * @param {String} action.type
 * @param {Number} action.id
 */
export let ui = (state, action) => {
    switch (action.type) {
        case 'ACTIVATE_LOCATION':
            state = state.set('activeLocationId', action.id);
            break;
    }

    return state;
};

/**
 * @param {Immutable} state
 * @param {Object} action
 * @param {String} action.type
 * @param {Number} action.id
 */
export let locations = (state, action) => {
    switch (action.type) {
        // @param {String} action.name
        case 'CHANGE_NAME_LOCATION':
            let locationIndex;

            locationIndex = state.findIndex(function (location) {
                return location.get('id') === action.id;
            });

            state = state.update(locationIndex, function (location) {
                return location.set('name', action.name);
            });
            break;
    }

    return state;
};
```

### `app.js`

```js
import React from 'react';

import {
    connect
} from 'react-redux';

/**
 * @param {Immutable}
 * @return {Object} state
 * @return {Object} state.ui
 * @return {Array} state.locations
 */
let selector = (state) => {
    state = state.toJS();

    // Selector logic ...

    return state;
};

class App extends React.Component {
    render () {
        return <div></div>;
    }
}

export default connect(selector)(App);
```
