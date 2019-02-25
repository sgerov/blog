+++
date = "2019-02-23"
title = "Redux by example"
description = "Learn the basics of redux by example. We will go through concepts like actions, reducers and the store in an incremental complexity fashion."
slug = "redux-by-example"
tags = [
    "redux", "react", "typescript"
]
categories = []
draft=true
+++

You will learn to: setup a new __react__ application containing listings of cars. Introduce the concept of an __action__, __reducer__ and __store__ with no external library and later add __redux__ on top of it. Data will be __hardcoded__ at first and come from an __async action__ later on.

## Setup

[Node.js](https://nodejs.org/) is the runtime environment for JS in your machine [(V8, as in Chrome)](https://v8.dev/). [npm](https://www.npmjs.com/) is the Node.js Package Manager which client will allow for more than just dependency management.

Assuming you already have __Node.js__ with __npm__ installed, let's scaffold a __React__ project [(using typescripts' starter)](https://github.com/Microsoft/TypeScript-React-Starter) using npm client:

{{< highlight bash >}}
# scaffold react + typescript app
npm init react-app ./redux-by-example --scripts-version=react-scripts-ts
{{< /highlight >}}

Now we have a React + Typescript scaffold in _redux-by-example_ folder. If you check on the dependencies within _package.json_ npm config file you will see just a few: 

- __react__
- __react-dom__ 
- __react-scripts-ts__

__react-scripts-ts__ is the responsible of hiding the complexity of setting up [Webpack](https://webpack.js.org/) (bundles all our Typescript code into executable JS code) and providing convenience scripts.


{{< highlight bash >}}
# start application in http://localhost:3000/
npm run start
{{< /highlight >}}

## UI and Data

Great! We have a React application running. Let's first start by defining the UI of our car listings. First of all, remove the test file to avoid compilation errors:

{{< highlight bash >}}
rm src/App.test.tsx
{{< /highlight >}}

Next lets define the shape of our application data with Typescript interfaces within _src/index.tsx_. 

We will have cars that contain a _model_ and a _price_. 

{{< highlight ts >}}
export interface ICar {
  model: string;
  price: number;
}
{{< /highlight >}}

Let's keep the state of the application in a centralised place which we call the __store__ and will contain an array of cars:

{{< highlight ts >}}
export interface IStore {
  cars: ICar[];
}
{{< /highlight >}}

Let's generate some example data and pass it through to our <App/> component:

{{< highlight jsx >}}
const initialState = { cars: [] }

// holds the state of the app
function createStore() : IStore {
  return initialState;
}
const store = createStore()

// define some cars (data could come from a user interaction)
const car1 = { model: 'BMW', price: 5400 }
const car2 = { model: 'Mercedes', price: 6000 }

// update the store
store.cars.push(car1, car2)

// pass the store to our component
ReactDOM.render(
  <App store={store}/>,
  document.getElementById('root') as HTMLElement
);
{{< /highlight >}}

Of course now our <App/> component should accept the store as a prop and print it:

{{< highlight jsx >}}
import { IStore } from './index';

class App extends React.Component<{ store: IStore }, any> {
  public render() {
    const { store: { cars: carsArray } } = this.props;
    return (
      <div>
        {carsArray.map( (car, index) => {
          return <h3 key={index}>{car.model} - {car.price}EUR<br/></h3>
        })}
      </div>
    )
  }
}
{{< /highlight >}}

which looks like:

![Redux hardcoded car listings](/images/redux-by-example-hardcoded.png)

Yay! Now we have a store which our component receives as a prop and get's its' data from. We can mutate the state of the store directly but you can guess how unmaintainable and error prone that would be. What if we could reason about every mutation that happened?

## Actions and Reducers

A way of modifying the store in a predictable way is by using actions and reducers:

- Actions: has a type (so we differentiate different ones) and a payload (in case we need data).
- Reducers: takes the current state and applies a transformation depending on the action type.

Let's modify our previous code to use an action and a reducer and you will better understand its' purpose:

{{< highlight jsx >}}

// we said an action has a type and a payload
export interface IAction {
  data: ICar;
  type: string;
}

// action creator to add a car
const createAction = (model: string, price: number) : IAction => {
  return {
    data: { model, price },
    type: 'ADD_CAR'
  }
}

// reducer that transforms the state given our action
const reducer = (state: IStore, action: IAction) : IStore => {
  switch (action.type) {
    case 'ADD_CAR':
      return {
        ...state,
        cars: [...state.cars, action.data]
      }
    default:
      return state;
  }
}

// define some cars (data could come from a user interaction)
let store = createStore()
store = reducer(store, createAction('BMW', 5400))
store = reducer(store, createAction('Mercedes', 6000))

// pass store to the component
{{< /highlight >}}

We have provided the reducer which is a __pure__ function that accepts a state and an action and returns a new state. That solves the issue of randomly modifying the state. Assuming the reducer is properly tested it would be impossible to get to an inconsistent state as we are just applying one transformation after the other.

You might have noticed we didn't introduce any third party library yet, and that's because the concept behind redux is not that complex. There are issues we haven't solved though, such as:

- How would a component re-render on state change?
- How would we call our reducer from a component without passing around the store? (specially for deep component trees)
- How do we debug all transitions from one state to another?

These and more issues are covered by the redux library.

## Redux
Let's first start by adding redux to the project:

{{< highlight bash >}}
# install redux
npm install react-redux

# install typescript definitions for development
npm install -D @types/react-redux
{{< /highlight >}}

Now that we have redux installed, lets hook its' store into our app:


{{< highlight jsx >}}
// delete our custom createStore function and import redux one:
import {createStore} from 'redux';

// change our reducer to have an initial state
const initialState = { cars: [] }
const reducer = (state: IStore = initialState, action: IAction) : IStore => {
// ...
}

// create the store this time passing our reducer as a parameter so redux can hook it
const store = createStore(reducer)

// dispatch our actions
store.dispatch(createAction('BMW', 5400))
store.dispatch(createAction('Mercedes', 60000))

// pass the current store state to our component
<App store={store.getState()}/>
{{< /highlight >}}

Now we are using the redux store to dispatch the actions we defined. Redux will take care of executing the actions in order and maintaining the state within the store.

We know how to dispatch actions and get the store state, but we haven't solved the issue of having access to the store nor dispatching actions from any component within the component tree. Let's see how Redux gets hooked.

## Hook redux
Subscribe to store changes and hook component to the store

## Async
Get data from an async source
