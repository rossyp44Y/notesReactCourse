> markdown: https://guides.github.com/features/mastering-markdown/  
> Preview: Chrome extension and QuickLook / space key on the file 

React Course - Udemy - Andrew Mead - Part II
============================================
<!-- TOC -->

- [React Router](#react-router)
  - [Install](#install)
  - [Import](#import)
  - [Use BrowserRouter and Router](#use-browserrouter-and-router)
  - [Webpack Configuration](#webpack-configuration)
  - [Setting Up a 404](#setting-up-a-404)
  - [Linking between Routes](#linking-between-routes)
  - [Static content across pages](#static-content-across-pages)
  - [Navigation panel](#navigation-panel)
  - [Organizing our Routes](#organizing-our-routes)
  - [Query Strings and URL Parameters](#query-strings-and-url-parameters)
- [Redux](#redux)
  - [The issues managing state](#the-issues-managing-state)
  - [How Redux works](#how-redux-works)
  - [Setting up Redux](#setting-up-redux)
  - [Dispatching Actions](#dispatching-actions)
  - [Subscribing and dynamic actions](#subscribing-and-dynamic-actions)
    - [Watch for changes to the store](#watch-for-changes-to-the-store)
    - [How to dispatch an action and pass some data along](#how-to-dispatch-an-action-and-pass-some-data-along)
  - [Action generators](#action-generators)
  - [Reducers](#reducers)
  - [Working with multiple Reducers](#working-with-multiple-reducers)
  - [Setting filters](#setting-filters)
  - [Sorting](#sorting)
- [ES6 features & Others](#es6-features--others)
  - [uuid](#uuid)
  - [time](#time)
  - [Object Destructuring](#object-destructuring)
  - [Array Destructuring](#array-destructuring)
  - [Function parameters destructuring](#function-parameters-destructuring)
  - [Array Spread Operator](#array-spread-operator)
  - [Object Spread Operator](#object-spread-operator)

<!-- /TOC -->

# React Router
- There are differences between server routing and client routing
- Client routing was already popular. React provide its implementation for this pattern
- https://reacttraining.com/react-router/
![Client Routing](./img/react-router.png)

## Install
- There are specific versions for web development and native. We'll use the web one
  ```
  yarn add react-router-dom
  ```

## Import
  ```javascript
  //app.js
  import { BrowserRouter, Route } from 'react-router-dom'
  ```

## Use BrowserRouter and Router
- By default the routing match is inclusive; so components that matches at least part of the ```path``` attribute will be rendered to the screen. That's the case of **/** which will be rendered alongside the other components because it includes the root. To avoid this we add the prop ```exact={true}``` to the ```Route```
  ```javascript
  const ExpenseDashboardPage = () => (<div>This is from Dashboard component</div>)
  const AddExpensePage = () => (<div>This is from AddExpense component</div>)
  
  const routes = (
    <BrowserRouter>
      <div> // BrowserRouter API only allows 1 child
        <Route path="/" component={ExpenseDashboardPage} exact={true} />
        <Route path="/create" component={AddExpensePage} />
      </div>
    </BrowserRouter>
  );

  ReactDOM.render(routes, document.getElementById('app'))
  ```

## Webpack Configuration
- Browsers by default will try to use server side routing resulting in 404 errors. We need to tweak devServer webpack config to send index.html back for all routes and let react-router figure out what to show in the screen
- Add ```historyApiFallback: true``` to webpack config to tell the devServer to always serve up the index.html file for all unknown 404s
  ```javascript
  devServer: {
    contentBase: path.join(__dirname, 'public') ,
    historyApiFallback: true
  }
  ```
- Restart dev-server every time the webpack config file is updated ```yarn run dev-server```

## Setting Up a 404
- ```Switch``` will go through all the different ```Route``` inside and will stop when it finds a match. The last one always matches and will show up for routes that aren't defined
  ```javascript
  import { BrowserRouter, Route, Switch } from 'react-router-dom'
  const routes = (
    <BrowserRouter>
      <Switch>
        <Route path="/" component={ExpenseDashboardPage} exact={true} />
        <Route path="/create" component={AddExpensePage} />
        <Route path="/edit" component={EditExpensePage} />
        <Route path="/help" component={HelpPage} />
        <Route component={NotFoundPage} />
      </Switch>
    </BrowserRouter>
  );

  ReactDOM.render(routes, document.getElementById('app'))
  ```

## Linking between Routes
- By default the browser is trying server-side routing. So, if we use a regular ```<a href="/">Go home </a>``` to link between pages we'll get a full refresh of the Page. We need to override this browser's default behavior. react-router provides ```<Link>``` and ```<NavLink>``` for us to implement client-side routing and achieve navigation between routes without refreshing the whole page
- By using client routing, Javascript just swaps things out on the fly and makes a new call to react to render the new component
  ```javascript
  import { BrowserRouter, Route, Switch, Link } from 'react-router-dom'
  const NotFoundPage = () => (
    <div>
      404! - <Link to="/">Go home</Link>
    </div>
  );
  ```
- If we change the url directly in the browser's address bar we'll still get full refresh
- If we need to navigate to an external URL, we use regular anchor tags ```<a href="/">ExternalPage</a>```, but if we need to navigate inside our app we use ```<Link to="">```

## Static content across pages
- We can have some static content that appears in all pages regardless the route. eg. Header and Footer. For this, we show the ```Header``` component before the ```Switch```. Remember that ```BrowserRouter``` allows only 1 child, so we go back to using a ```<div>``` root
  ```javascript
  const Header = () => (
    <header>
      <h1>Expensify</h1>
    </header>
  );
  const routes = (
    <BrowserRouter>
      <div>
        <Header />
        <Switch>
          <Route path="/" component={ExpenseDashboardPage} exact={true} />
          <Route path="/create" component={AddExpensePage} />
          <Route component={NotFoundPage} />
        </Switch>
      </div>
    </BrowserRouter>
  );
  ```

## Navigation panel
- react-router provides an alternative to Link, especially for navigation panels, allows by default to highlight the current page ```<NavLink to="">```
- The prop ```to``` follows the same matching rule as Route, so we add the exact parameter
  ```javascript
  const Header = () => (
    <header>
      <h1>Expensify</h1>
      <NavLink to="/" activeClassName="is-active" exact={true}>Dashboard</NavLink>
      <NavLink to="/create" activeClassName="is-active">Create Expense</NavLink>
      <NavLink to="/edit" activeClassName="is-active">Edit Expense</NavLink>
      <NavLink to="/help" activeClassName="is-active">Help</NavLink>
    </header>
  );
  ```
  ```scss
  .is-active {
    font-weight: bold;
    margin-right: 3rem;
    color: orange;
  }
  header a {
    margin-right: 3rem;
  }
  ```
## Organizing our Routes
- Each individual component is better placed inside its own file including Header and NotFoundPage.
  ```javascript
  //file: src/components/NotFoundPage.js
  import React from "react";
  import { Link } from "react-router-dom";

  const NotFoundPage = () => (
    <div>
      404! - <Link to="/">Go home</Link>
    </div>
  );

  export default NotFoundPage

  //file: src/components/AddExpensePage.js
  import React from "react";

  const AddExpensePage = () => (
    <div>
      This is from AddExpense component
    </div>
  );

  export default AddExpensePage
  ```
- The router is also better placed as a functional component in a different folder and in here import all other components
  ```javascript
  //file: src/routes/AppRouter.js
  import React from "react";
  import { BrowserRouter, Route, Switch } from "react-router-dom";
  import Header from '../components/Header'
  import ExpenseDashboardPage from '../components/ExpenseDashboardPage'
  import AddExpensePage from "../components/AddExpensePage";
  import EditExpensePage from "../components/EditExpensePage";
  import HelpPage from "../components/HelpPage";
  import NotFoundPage from "../components/NotFoundPage";

  const AppRouter = () => (
    <BrowserRouter>
      <div>
        <Header />
        <Switch>
          <Route path="/" component={ExpenseDashboardPage} exact={true} />
          <Route path="/create" component={AddExpensePage} />
          <Route path="/edit" component={EditExpensePage} />
          <Route path="/help" component={HelpPage} />
          <Route component={NotFoundPage} />
        </Switch>
      </div>
    </BrowserRouter>
  );

  export default AppRouter
  ```
- app.js remains as a simple file where we render AppRouter component.
  ```javascript
  //file: src/app.js
  import React from 'react'
  import ReactDOM from 'react-dom'
  import AppRouter from './routers/AppRouter'
  import 'normalize.css/normalize.css'
  import './styles/styles.scss'

  ReactDOM.render(<AppRouter />, document.getElementById('app'))
  ```

## Query Strings and URL Parameters
- When react-router finds a route that is a match, it renders an instance of that component. No only is it rendering the component it's also passing a few props down. So anytime we use a component inside a route we get access to some special information.
  - history: things like goBack, goForward, push, replace - allow us to manipulate where the user is directed
  - match: contains param object for dynamic urls
  - location: information about the current url
    ```
    localhost:8080/edit?query=rent&sort=date
    search will bring: ?query=rent&sort=date
    we can also get the hash value
    ```
- Our App will have paths like ```localhost:8080/edit/107``` meaning that we intend to edit the expense with id 107. So we need to be able to set an manipulate these dynamic urls
  - We need to tweak the Route. Add colon ```(:)``` and what comes after is a variable name. This is going to dynamically match whatever comes after the forward slash in the url so that we can do something meaningful with it like fetching the item from the database
    ```javascript
    <Route path="/edit/:id" component={EditExpensePage} />
    ```
  - Now the value in the url (107) comes in the variable **id** under props.match.params

# Redux
- Redux is a state management library that integrates nicely with React
- Allow us to track changing data
- Component's state and redux they both aim to solve the same problem which is to manage the state for your application - tracking changing data
- State is data that changes. For components' state we used ```this.setState``` to change data and ```this.state.value``` to read some value from there

## The issues managing state
- When having complex Apps there's no way to share state across all components
- For Simple Apps we can keep the state in a Parent component but for Complex Apps where to keep the state and how to share it? we don't have a Parent, instead a Router decide which component to show on screen - Complex State is Complex
![Complex state](./img/complex-state.png)
- Components in Apps like IndecisionApp are closely bound, we break things in Components which is good but the child component need props passed down from the parent so the component can't really be reused wherever we want. We will need then pass down some props to components that don't even need to know about them - Components really aren't reusable
- What we need is a way for components to be able to interact with the application state both getting and setting value without having to manually pass props down to the entire component tree. It would be much nicer if each component could describe what it needs from the state and what it needs to be able to change on this state
![Reusable](./img/reusable.png)
- Some questions emerge
  - Where do I store my app state in a complex React app?
  - How do I create components that are actually reusable?
- The response is: use Redux
- There's no wrong using ```props``` it's a perfectly valid way of communicating data between parent and child eg. Options to Option in IndecisionApp or Expenses to Expense in Expensify
- The scenario where we really want to avoid ```props``` is when we're passing props down this long chain of components just to get it to the last one in the chain. The components in between aren't actually using the value, they're just passing it along. In that case we want to avoid using props and we should use something like **Redux** instead

## How Redux works
- Redux is a state container same as our class based components
- There's an object called **Redux Store** we can read and change
- Individual components are gonna be able to determine how they want to read and change the elements on the store
![Redux](./img/redux.png)

## Setting up Redux
- https://redux.js.org
- Change Webpack to read a different file
  ```javascript
  entry: './src/playground/redux-101.js',
  ```
- Install Redux ```yarn add redux```
- Restart dev server ```yarn run dev-server```
- Create redux state container
  ```javascript
  import { createStore } from 'redux'

  //first parameter is the current state
  //we don't have a constructor function where we can set up the default so we do it inline
  //if there is not current state what's the default object?
  const store = createStore((state = { count: 0 }) => { //current state
    return state
  })

  console.log(store.getState()) //output: {count: 0}
  ```
- We have to pass a function to ```createStore()``` and that function gets called once right away. There's no state the first time redux calls this function so the default state value is used. We then return that and it becomes the new state
- We can fetch the current state using the ```getState()``` method on the store; this get the actual object and we can use the values inside for our purposes eg. render a component etc
- Now, how to increment the count? or how to reset the count to zero? - Actions is the response

## Dispatching Actions
- Actions allow us to change the redux store
- An action is nothing more than an object that gets sent to the store
- This object describes the **type of action** we'd like to take. So we'll have actions like increment, decrement and reset. This is going to allow us to change the store over time by just dispatching various actions
- The object will contain a property ```type``` and the value by convention is all uppercase and underscore if more than one word
- Then we send the object to the store using ```store.dispatch({ my_action_object })```
- When we call ```store.dispatch``` the function ```createStore``` runs for the second time. The action object gets passed as the second argument to the function 
- Then we can use the action type to take action like adding one to the count otherwise it's the first time Redux is running the function and just return the state without change
- Inside the action type checking we return an object with the new state, we shouldn't change the state or action but instead return the new state we use the state values to compute the new state - similar to this.setState
  ```javascript
  import { createStore } from 'redux'

  const store = createStore((state = { count: 0 }, action) => { //current state
    if (action.type === 'INCREMENT') {
      return {
        count: state.count + 1 //return new state object
      }
    } else {
      return state //return state unchanged
    }
  })

  console.log(store.getState()) //output: {count: 0}

  // I'd like to increment the count
  store.dispatch({ //dispatch an action of type INCREMENT to change the store
    type: 'INCREMENT' //uppercase by convention
  })

  console.log(store.getState()) //now state has changed and output is {count: 1}
  ```
- It's a common pattern to use a ```switch``` statement instead of if/else because it's easier to read and scale
  ```javascript
  const store = createStore((state = { count: 0 }, action) => {
    switch (action.type) {
      case 'INCREMENT':
        return {
          count: state.count + 1 //return new state object
        }
      case 'DECREMENT':
        return {
          count: state.count - 1
        }
      case 'RESET':
        return {
          count: 0
        }
      default: //runs if none of the other cases run
        return state; //return state unchanged
    }
  })

  // dispatch actions
  store.dispatch({
    type: 'INCREMENT'
  })
  store.dispatch({
    type: "DECREMENT"
  })
  store.dispatch({
    type: 'RESET'
  })
  ``` 

## Subscribing and dynamic actions

### Watch for changes to the store
- We need to be aware of changes to the store in order to re-render our application or do something of interest to our application
- Redux gives us ```store.subscribe``` and we pass a function that's gonna be called every single time the store changes 
  ```javascript
  store.subscribe(() => {
    console.log(store.getState()) //gets called every time an action is dispatched twice in this case
  })
  store.dispatch({
    type: 'INCREMENT' 
  })
  store.dispatch({
    type: "INCREMENT"
  });
  ```
- We can also remove the subscription by storing the return from subscribe and calling it whenever we want to unsubscribe
  ```javascript
  const unsubscribe = store.subscribe(() => {
    console.log(store.getState()) //gets called only once because we unsubscribe after the first dispatch
  })
  store.dispatch({
    type: 'INCREMENT' 
  })
  unsubscribe()
  store.dispatch({
    type: "INCREMENT"
  });
  ```

### How to dispatch an action and pass some data along 
- We might want to pass some dynamic information to the action for instance user input
- `type` is mandatory in all actions otherwise Redux is going to crash but we can also toss as many additional things as you want
- For instance we might want to pass an **optional** property ```incrementBy``` and consider it in the Action handler. We will have access to this property through action.incrementBy in a similar way as we get access to action.type
  ```javascript
  const store = createStore((state = { count: 0 }, action) => {
    switch (action.type) {
      case 'INCREMENT':
        //if there's a value take it otherwise default it to 1
        const incrementBy = typeof action.incrementBy === 'number' ? action.incrementBy : 1
        return {
          count: state.count + incrementBy 
        }
      default: //runs if none of the other cases run
        return state; //return state unchanged
    }
  })

  store.dispatch({
    type: 'INCREMENT',
    incrementBy: 5 
  })
  store.dispatch({
    type: "INCREMENT"
  });
  ```
- We can also make the additional data **mandatory** by just forcing it to be passed to get the desired data. If not passed we won't have an error but it will be undefined leading to wrong behavior
  ```javascript
  const store = createStore((state = { count: 0 }, action) => {
    switch (action.type) {
      case 'SET':
        return {
          count: action.count
        }
      default: //runs if none of the other cases run
        return state; //return state unchanged
    }
  })
  store.dispatch({
    type: 'SET',
    count: 101
  })
  ```

## Action generators
- Functions that return Action Objects
- All action will be created in one place and we will have a function we can call to generate the action objects
- This way our code is less error prone especially because action types are just capitalized strings and easy to make a typo. Having a method we can call help us to have better code
- We we're going to be preferring action generators over inline action objects
  ```javascript
  //inline action objects
  store.dispatch({
    type: "INCREMENT"
  });
  ```
  ```javascript
  //action generators
  //long way
  const incrementCount = () => {
    return {
      type: 'INCREMENT'
    }
  }
  //or shorthand implicitly return object - need () wrapping the object
  const incrementCount = () => ({
    type: 'INCREMENT'
  })

  //call
  store.dispatch(incrementCount());
  ```
- We can also pass data to action generators and this data is commonly referred as payload
  ```javascript
  //action generator
  const incrementCount = (payload = {}) => ({ //default to empty object to avoid trying to get a property out of undefined
    type: 'INCREMENT',
    incrementBy: typeof payload.incrementBy === 'number' ? payload.incrementBy : 1
  })
  const store = createStore((state = { count: 0 }, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return {
        count: state.count + action.incrementBy 
      }
  ...
  }
  store.dispatch(incrementCount({ incrementBy: 5 }));
  ```
- Now, we can take it a step further and simplify the action generator by using destructuring 
  ```javascript
  //instead of using the payload object coming in, we can destructure it and take incrementBy value
  const incrementCount = ( { incrementBy } = {}) => ({ 
    type: 'INCREMENT',
    incrementBy: typeof incrementBy === 'number' ? incrementBy : 1
  })
  //then when we destructure we know we can set up default values. so if incrementBy exists use it otherwise use 1, now we don't need the ternary operator
  const incrementCount = ( { incrementBy = 1 } = {}) => ({ 
    type: 'INCREMENT',
    incrementBy: incrementBy
  })
  //we also know that when we're setting an object property equal to a variable name with the same name we can simplify it
  const incrementCount = ( { incrementBy = 1 } = {}) => ({ 
    type: 'INCREMENT',
    incrementBy
  })
  ```

## Reducers
- Actions describe the fact that something happened, but don't specify how the application's state changes in response. This is the job of reducers
- Reducers are pure functions
  - output is only determined by the input. ```state``` and ```action``` in this case. it doesn't use anything else from outside of the function scope and it doesn't change anything outside
  - Reducers need to compute the new state only based on the old/current state and the action
- Never change ```state``` or ```action```. instead we read those two values and return a new state object. mutating the state might bring undesired effects
  ```javascript
  //Reducers
  const countReducer = (state = { count: 0 }, action) => {
    switch (action.type) {
      case "INCREMENT":
        return {
          count: state.count + action.incrementBy
        };
      case "DECREMENT":
        return {
          count: state.count - action.decrementBy
        };
      case "RESET":
        return {
          count: 0
        };
      case "SET":
        return {
          count: action.count
        };
      default:
        //runs if none of the other cases run
        return state; //return state unchanged
    }
  };

  const store = createStore(countReducer)
  ```

## Working with multiple Reducers
- When handling several pieces of state we'll have many actions and a single reducer is just not feasible 
- We can split the reducers for each root property in our redux store
- For example, actions related to handling the expenses in one reducer and actions related to filters in another reducer. Then we have to combine them together to create the complete store
- We use combineReducers from 'redux' to bring them together
  ```javascript 
  /* Actions
  // ADD_EXPENSE
  // REMOVE_EXPENSE
  // EDIT_EXPENSE
  // SET_TEXT_FILTER
  // SORT_BY_DATE
  // SORT_BY_AMOUNT
  // SET_START_DATE
  // SET_END_DATE */
  
  // Expenses Reducer
  const expensesReducerDefaultState = []
  const expensesReducer = (state = expensesReducerDefaultState, action) => {
    switch (action.type) {
      default:
        return state  
    }
  }

  // Filters Reducer
  const filtersReducerDefaultState = {
    text: '',
    sortBy: 'date',
    startDate: undefined,
    endDate: undefined
  }
  const filtersReducer = (state = filtersReducerDefaultState, action) => {
    switch (action.type) {
      default: 
        return state
    }
  }

  // Store creation
  const store = createStore(
    combineReducers({
      expenses: expensesReducer,
      filters: filtersReducer
    })
  )
  console.log(store.getState())
  ```
- When we dispatch an action it will be dispatched to both reducers. We set switch/case in the reducer that actually needs to handle that specific action. We also need to set up the action generator
  ```javascript
  // ADD_EXPENSE - Action generator
  const addExpense = (
    {description = '', note = '', amount = 0,  createdAt = 0} = {} // default values
  ) => ({ //implicitly object return
    type: 'ADD_EXPENSE',
    expense: {
      id: uuid(),
      description,
      note,
      amount,
      createdAt
    }
  })

  // Expenses Reducer - add case to handle ADD_EXPENSE
  const expensesReducerDefaultState = []
  const expensesReducer = (state = expensesReducerDefaultState, action) => {
    switch (action.type) {
      case 'ADD_EXPENSE': 
        return state.concat(action.expense) //we shouldn't change state so we use concat instead of push. concat returns a new array 
      default:
        return state  
    }
  }

  // Filters Reducer
  const filtersReducerDefaultState = {
    text: '',
    sortBy: 'date',
    startDate: undefined,
    endDate: undefined
  }
  const filtersReducer = (state = filtersReducerDefaultState, action) => {
    switch (action.type) {
      default: 
        return state
    }
  }

  // Store creation
  const store = createStore(
    combineReducers({
      expenses: expensesReducer,
      filters: filtersReducer
    })
  )

  store.subscribe(() => {
    console.log(store.getState()); //listen to changes in the state
  })

  store.dispatch(addExpense({ description: 'Rent', amount: 100 })) //action gets dispatched to both reducers - amount expressed in pennies $1 = 100pennies
  ```
- As an alternative to ```concat``` we can use the spread operator which basically takes whatever it contained before and adds more data as instructed
  ```javascript
  //the two return statements are equivalent
  switch (action.type) {
  case 'ADD_EXPENSE': 
    return state.concat(action.expense) 
  }
  switch (action.type) {
  case 'ADD_EXPENSE': 
    return [
      ...state, //spread whatever was in variable state
      action.expense  //and add this element
    ]
  }
  ```
- We can also use spread operator to manage complex objects inside our app
  ```javascript
  // EDIT_EXPENSE - Action generator
  const editExpense = (id, updates) => ({ //takes the id and an object with things to update
    type: 'EDIT_EXPENSE',
    id,
    updates
  })
  // Reducer
  case 'EDIT_EXPENSE':
    return state.map((expense) => { //map iterates through every item of the array and returns a new array
      if (expense.id === action.id) { //id id is the one we want to edit
        return {
          ...expense, //takes the object
          ...action.updates //adds the things to update
        }   
      } else { // this is actually optional
        return state
      }
    })
  // Store creation  
  // Action
  const expenseTwo = store.dispatch(addExpense({ description: "Coffee", amount: 300 }));
  store.dispatch(editExpense(expenseTwo.expense.id, { amount: 500 }));
  ```
- The spread operator for filters
  ```javascript
  // SET_TEXT_FILTER
  const setTextFilter = (text = '') => ({
    type: 'SET_TEXT_FILTER',
    text
  })
  // Filters Reducer
  const filtersReducerDefaultState = {
    text: '',
    sortBy: 'date',
    startDate: undefined,
    endDate: undefined
  }
  const filtersReducer = (state = filtersReducerDefaultState, action) => {
    switch (action.type) {
      case 'SET_TEXT_FILTER':
        return {            //new object
          ...state,         //take the state values - clone
          text: action.text //override the property text
        }
      default: 
        return state
    }
  }
  // Store creation  
  // Action
  store.dispatch(setTextFilter('rent'))
  store.dispatch(setTextFilter())
  ```

## Setting filters
  ```javascript
  // Get visible expenses
  const getVisibleExpenses = (expenses, { text, sortBy, startDate, endDate }) => {
    return expenses.filter((expense) => {
      const startDateMatch = typeof startDate !== 'number' || expense.createdAt >= startDate
      const endDateMatch = typeof endDate !== 'number' || expense.createdAt <= endDate
      const textMatch = expense.description.toLowerCase().includes(text.toLowerCase())

      return startDateMatch && endDateMatch && textMatch
    }) 
  }
  store.subscribe(() => {
    const state = store.getState()
    const visibleExpenses = getVisibleExpenses(state.expenses, state.filters)
    console.log(visibleExpenses);
  })
  //actions
  const expenseOne = store.dispatch(addExpense({ description: 'Rent', amount: 100, createdAt: 1000 }))
  const expenseTwo = store.dispatch(addExpense({ description: 'Coffee', amount: 300, createdAt: -1000 }));
  //filters
  store.dispatch(setTextFilter('rEnT'))
  store.dispatch(setStartDate(-2000))
  store.dispatch(setEndDate(2000));
  ```

## Sorting
  ```javascript
  // Get visible expenses
  const getVisibleExpenses = (expenses, { text, sortBy, startDate, endDate }) => {
    return expenses.filter((expense) => {
      const startDateMatch = typeof startDate !== 'number' || expense.createdAt >= startDate
      const endDateMatch = typeof endDate !== 'number' || expense.createdAt <= endDate
      const textMatch = expense.description.toLowerCase().includes(text.toLowerCase())

      return startDateMatch && endDateMatch && textMatch
    }).sort((a, b) => {
      if (sortBy === 'date') {
        return a.createdAt < b.createdAt ? 1 : -1
      } else if (sortBy === 'amount') {
        return a.amount < b.amount ? 1 : -1
      }
    }) 
  }
  ```


# ES6 features & Others

## uuid
```javascript
yarn add uuid //install
import uuid from 'uuid' //import
const expense = { //use
  id: uuid(),
  description: ''
}
```

## time
- timestamps (milliseconds)
- January 1st 1970 (unix epoch)
- 33400, 10, -203
- timezone independent time data

## Object Destructuring
- Allow us to pull out properties off of an object instead of having to pass the entire object. Leads to easier to read and maintain code 
  ```javascript
  const person = {
    name: 'Rocio',
    age: 38,
    location: {
      city: 'Melbourne',
      temp: 29
    }
  }

  //basic destructing
  const { name, age } = person
  console.log(`${name} is ${age}.`)
  //This is equivalent to
  console.log(`${person.name} is ${person.age}.`);

  //rename
  const { city, temp: temperature } = person.location
  if (city && temperature) {
    console.log(`It's ${temperature} in ${city}`);
  }

  //rename and default value (if there isn't one)
  const { name: firstName = 'Anonymous' } = person;
  console.log(`Hi ${firstName}.`);
  ```
- Nested objects
  ```javascript
  //first way
  const book = {
    title: 'Ego is the Enemy',
    author: 'Ryan Holiday',
    publisher: {
      name: 'Penguin'
    }
  }

  const { name: publisherName = 'Self-Published' } = book.publisher
  console.log(publisherName)

  //second way - combined
  const book2 = {
    title: "Ego is the Enemy",
    author: "Ryan Holiday",
    publisher: {
      name: "Penguin"
    }
  };

  const { title, publisher: { name: publisherName2 = "Self-Published" } } = book2;
  console.log(`${title} by ${publisherName2}`);
  ```

## Array Destructuring
- We can also pull oit values off an array
  ```javascript
  const address = ['123 Main St', 'Chicago', 'Illinois', '60601']
  const [ , city, state ] = address //matches by position/index
                                    //skip the 1st take 2nd and 3rd and stop

  console.log(`You are in ${city} ${state}.`);
  // equivalent to
  console.log(`You are in ${address[1]} ${address[2]}.`)

  //no renaming
  //default values
  const address2 = [];
  const [, , state2 = 'New York'] = address2; //matches by position/index
  //skip the 1st take 2nd and 3rd and stop

  console.log(`You are in ${state2}.`);
  ```

## Function parameters destructuring
  ```javascript
  const todo = {
    id: "123",
    text: "Pay the bills",
    completed: false
  };
  const printTodo = ({ text, completed }) => {
    console.log(`${text}: ${completed}`);
  };
  printTodo(todo);
  ```
  ```javascript
  const add = ({ a, b }, c) => {
    return a + b + c
  }
  console.log(add({ a: 1, b: 12 }, 100))
  ```

## Array Spread Operator
- We can spread the content of an array and add more values
  ```javascript
  let cities = ['Barcelona', 'Cape Town', 'Bordeaux'] 
  let citiesClone = [...cities, 'Santiago'] 
  console.log(cities) // Will print three cities 
  console.log(citiesClone) // Will print four cities
  ```

## Object Spread Operator
- We need to add a plugin to our babel configuration
```javascript
yarn add babel-plugin-transform-object-rest-spread
```
- Add to .babelrc file in the plugins array
  ```javascript
  "plugins": [
    "transform-class-properties",
    "transform-object-rest-spread"
  ]
  ```
- Use the Spread Operator with objects. It's very useful because we don't want to modify state instead we want to clone it and add/override some properties
```javascript
const user = {
  name: 'Jen',
  age: 24
}
//cloning an object - output: {name: "Jen", age: 24}
console.log({
  ...user
})
//define new properties - output: {name: "Jen", age: 24, location: "Melbourne"}
console.log({
  ...user,
  location: 'Melbourne'
})
//override existing properties - output: {name: "Jen", age: 38, location: "Melbourne"}
//the position of the overriding is important
console.log({
  ...user,
  location: 'Melbourne',
  age: 38
});
```

