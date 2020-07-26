# Intro to Redux: Reading Data from State

## Objectives

- Use the `createStore()` method provided by the redux library.

## Introduction

In the previous section, we used a **createStore()** method that we wrote, 
and passed a reducer to it. We used the **dispatch** method from the store 
to dispatch actions and update the state.

Now let's think about which part of our application would belong in the official
Redux library -- that is, which part of our codebase would be common to all
applications. Well, probably not the reducer as our reducers seem unique to each
React & Redux application. The reducers are unique because sometimes we have
reducers that would add or remove items, or add or remove users, or edit users,
etc. What these actions are and how the reducer manages the state is customized.
Thus, the reducer would not be part of the Redux library that other developers
would use to build their application.

The **createStore()**, method however is generic across Redux applications. It
always returns a store (given a reducer) that will have a dispatch method and a
getState method.

So from now on, we will import our **createStore()** method from the official
Redux library. Normally, to install Redux into a React application, you need to
install two packages, `redux` and `react-redux`, by running 
`npm install redux && npm install react-redux`. These are already included in 
this lesson's `package.json` file, so all you need to do is run 
`npm install && npm start` to get started.

In this code along, we'll be building a simple shopping list application that
will allow a user to view an existing shopping list.

### Step 1: Setting Up The Store

First things first, we'll use Redux to initialize our store and pass it down to
our top-level container component.

Redux provides a function, `createStore()`, that, when invoked, returns an
instance of the Redux store for us. We want to import `createStore()` in our 
`src/index.js` file, where ReactDOM renders our application, and then use
that function to create the store.

```javascript
// ./src/index.js

import React from 'react';
import ReactDOM from 'react-dom';
import { createStore } from 'redux'; /* code change */
import shoppingListItemReducer from './reducers/shoppingListItemReducer.js';
import App from './App';
import './index.css';

const store = createStore(shoppingListItemReducer); /* code change */

ReactDOM.render(<App />, document.getElementById('root'));
```

Now, with the above set up, let's pass `store` down to App as a prop, where we 
will then be able to access the **Redux** store.

```javascript
// ./src/index.js

import React from 'react';
import ReactDOM from 'react-dom';
import { createStore } from 'redux';
import shoppingListItemReducer from './reducers/shoppingListItemReducer.js';
import App from './App';
import './index.css';

const store = createStore(shoppingListItemReducer);


ReactDOM.render(
    <App store={store} />  /* code change */,
  document.getElementById('root')
);
```

So, to recap, just like we did previously, we call our **createStore()** method
in `src/index.js`. We pass our **createStore()** method a reducer, and then we
pass our newly created store to our **App** component as a prop. You can find
the reducer in `./src/reducers/shoppingListItemReducer.js`:

```javascript
// ./src/reducers/shoppingListItemReducer.js

export default function shoppingListItemReducer(
  state = {
    items: []
  },
  action
) {
  switch (action.type) {
    case 'INCREASE_COUNT':
      return {
        ...state,
        items: state.items.concat(state.items.length + 1)
      }

    default:
      return state;
  }
}
```

Ok so effectively, our reducer is just producing a counter. It adds a new item
to the list each time it is called, and that item is one more than the last
item.

Instead of having all of our functions encapsulated in a closure within
`index.js` as we did while building our own Redux set up, we've now separated
out the reducer function, giving it a relevant name, `shoppingListItemReducer`,
and let the Redux library take care of our `createStore` function. These two
pieces are both imported into `src/index.js` and used to create `store`.

Once we've created the store and passed it to the `App` component as a prop, 
we can access it using `this.props.store`:

```javascript
// ./src/App.js
import React, { Component } from 'react';
import './App.css';

class App extends Component {
	handleOnClick = () => {
		this.props.store.dispatch({
		  type: 'INCREASE_COUNT',
		});
	  }

	render() {
		const state = this.props.store.getState();
		return (
			<div className="App">
				<button onClick={this.handleOnClick}>Click</button>
				<p>{state.items.length}</p>
			</div>
		);
	}
}

export default App;
```

As you recall, the store contains two methods: `dispatch` and `getState`. We
use the `getState` method in our `render` method to get the current state so 
we can display it on the page. We also have an event handler that calls the 
`dispatch` method, passing in our action, when the button is clicked. 

Now, if you boot up the app, you should see a button on the page, followed by 
a zero. Then, if you click on the button... nothing happens. So what's gone
wrong here? Well, we haven't yet done all the work necessary to get our 
**React** and **Redux** libraries communicating with each other properly so 
the page re-renders when the state is updated. We'll tackle that in the next 
lesson. In the meantime, let's get some feedback.


#### Add Logging to Our Reducer

First, let's log our action and the new state. So we'll change the reducer to the following:

```javascript
// ./src/reducers/shoppingListItemReducer

export default function shoppingListItemReducer(
  state = {
    items: []
  },
  action
) {
  console.log(action);
  switch (action.type) {
    case 'INCREASE_COUNT':
      console.log('Current state.items length %s', state.items.length);
      console.log('Updating state.items length to %s', state.items.length + 1);
      return {
        ...state,
        items: state.items.concat(state.items.length + 1)
      };

    default:
      console.log('Initial state.items length: %s', state.items.length);
      return state;
  }
}
```

Ok, so this may look like a lot, but really all were doing is adding some
logging behavior. At the top of the function, we are logging the action. After
the case statement, we are storing our state as current state first. Then we are
logging the updating state value. Then under the default case statement, we
just can log the previous state because this state is unchanged.

Now, refresh your app, and give it a shot. You should see the correct action
being dispatched, as well as an update to the state. While we aren't getting our
state directly from the store, we know that we are dispatching actions. We know
this because each time we click a button, we call this.props.store.dispatch({ type:
'INCREASE_COUNT' }) and somehow this is hitting our reducer. So things are
happening.

#### Redux DevTools

There is this amazing piece of software that allows us to nicely view the state
of our store and each action that is dispatched. The software does a lot more
than that. I'll let you read about it here:
[redux-devtools-extension][devtools]. Ok, so let's get to incorporating this. In
fact, every time we use the Redux library going forward, we should make sure we
incorporate devtools. Otherwise, you are flying blind.

First, just Google for Redux Devtools Chrome. There you will find the Chrome
extension for Redux. Please download it, and refresh Chrome. To verify that you
have successfully installed the extension, go to your developer console in Google
Chrome (press command+shift+c to pull it up). In the top bar you will see a couple 
of arrows. Click those arrows, and if you see Redux in the dropdown, you have 
properly installed the Chrome extension. Step one is done.

Second, we need to tell our application to communicate with this extension.
Doing so is pretty easy. Now we change the arguments to our createStore method
to the following:

```javascript
// ./src/index.js

import React from 'react';
import ReactDOM from 'react-dom';
import { createStore } from 'redux';
import shoppingListItemReducer from './reducers/shoppingListItemReducer';
import App from './App';
import './index.css';

const store = createStore(
  shoppingListItemReducer,
  window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
); /* code change */

ReactDOM.render(
  <App store={store} />,
  document.getElementById('root')
);
```

Ok, notice that we are still passing through our reducer to the createStore
method. The second argument is accessing our browser to find a method called
`__REDUX_DEVTOOLS_EXTENSION__`. Now let's open the Redux Devtools (press 
command+shift+c, click on the arrows at the top right, and select the extension 
in the dropdown). Now click on the tab that says state. You should see 
`{ items: [] }`. If you do, it means that your app is now communicating with 
the devtool. Click on the button in your application, to see if the state 
changes. For each time you click on it, you should see the action name 
(`INCREASE_COUNT`) and the updated state show up in the devtools.

Whew!

### Summary

In this lesson, we saw how to use the **createStore()** method. We saw that we
can rely on the Redux library to provide this method, and that we still need to
write our own reducer to tell the store what the new state will be given a
particular action. We saw that when using the **createStore()** method, and
passing through a reducer, we are able to change the state just as we did
previously. We were able to see these changes by hooking our application up to a
Chrome extension called Redux Devtools, and then providing the correct
configuration.

[devtools]: https://github.com/zalmoxisus/redux-devtools-extension
