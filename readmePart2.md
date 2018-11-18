> markdown: https://guides.github.com/features/mastering-markdown/  
> Preview: Chrome extension and QuickLook / space key on the file 

React Course - Udemy - Andrew Mead - Part II
============================================
[[TOC]]

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
        return expense
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

# Reorganizing the App
- Move actions to src/actions/expenses.js and src/actions/filters.js, set exports
- Move reducers to src/reducers/expenses.js and src/reducers/filters.js, set exports
- Move function ```getVisibleExpenses``` to src/selectors/expenses.js, set exports
- Move Redux Store creation as a function to src/store/configureStore.js, set exports
- Now in src/app.js we can use all of them
  ```javascript
  import React from 'react'
  import ReactDOM from 'react-dom'
  import AppRouter from './routers/AppRouter'
  import configureStore from './store/configureStore'
  import { addExpense } from "./actions/expenses";
  import { setTextFilter } from "./actions/filters";
  import getVisibleExpenses from "./selectors/expenses";

  import 'normalize.css/normalize.css'
  import './styles/styles.scss'

  const store = configureStore()

  store.dispatch(addExpense({ description: 'Water bill', amount: 8700 }))
  store.dispatch(addExpense({ description: 'Gas bill', amount: 5000 }))
  store.dispatch(setTextFilter('bill'))

  const state = store.getState()
  const visibleExpenses = getVisibleExpenses(state.expenses, state.filters);
  console.log(visibleExpenses);

  ReactDOM.render(<AppRouter />, document.getElementById('app'))
  ```

# The Higher Order Components
- It's a pattern heavily used in libraries such as react-redux which allow us to connect our redux stores to our react components 
- It's just a component (the HOC) that renders another component (Regular Components)
- We might have several of the regular components and want to use the same higher component for all of them allowing us to reuse code (eg. disclaimer at the end of each medical card)
- The advantages of using HOC are
  - Reuse code
  - Render hijacking 
  - Prop manipulation
  - Abstract state
- HOC Example - show a disclaimer for Private Info
  ```javascript
  import React from 'react'
  import ReactDOM from 'react-dom'

  const Info = (props) => ( //Regular Component
    <div>
      <h1>Info</h1>
      <p>The info is: {props.info}</p>
    </div>
  )

  //regular function
  //gets called with the component we want to wrap
  //generic parameter name because it's going to be reused - uppercase because it's a component
  const withAdminWarning = (WrappedComponent) => { 
    //this is the HOC
    return (props) => (
      <div>
        {props.isAdmin && <p>This is private info. Please do not share!</p>}
        <WrappedComponent {...props} /> {/* pass props to the child component */}
      </div>
    ) 
  }
  //what we get back from withAdminWarning is an alternative version of <Info /> - the HOC
  const AdminInfo = withAdminWarning(Info)
  ReactDOM.render(<AdminInfo isAdmin={true} info="These are the details" />, document.getElementById('app'))
  ```
- HOC Challenge
  ```javascript
  import React from 'react'
  import ReactDOM from 'react-dom'

  const Info = (props) => (
    <div>
      <h1>Info</h1>
      <p>The info is: {props.info}</p>
    </div>
  )

  // requireAuthentication
  const requireAuthentication = (WrappedComponent) => {
    return (props) => (
      <div>
        {props.isAuthenticated 
          ? <WrappedComponent {...props} />
          : <p>Please login to view the info.</p>
        }
      </div>
    )
  }
  const AuthInfo = requireAuthentication(Info);
  ReactDOM.render(<AuthInfo isAuthenticated={false} info="These are the details" />, document.getElementById('app'))
  ```

# React-Redux

## Install and hook up store
- Install ```yarn add react-redux```
- To integrate Redux in our application we're going to be using the ```<Provider />``` component once at the root of our application and we're going to be using ```connect()``` for every single component that needs to connect to the redux store
- ```<Provider>``` allow us to provide the store to all of the components that make up our application so we don't need to manually pass the store around instead individual components that want to access the store can just access it
  ```javascript
  import { Provider } from 'react-redux'
  //.....more imports
  const store = configureStore()

  store.dispatch(addExpense({ description: 'Water bill', amount: 8700 }))
  store.dispatch(addExpense({ description: 'Gas bill', amount: 5000 }))
  store.dispatch(setTextFilter('bill'))

  const state = store.getState()
  const visibleExpenses = getVisibleExpenses(state.expenses, state.filters);
  console.log(visibleExpenses);

  const jsx = (
    <Provider store={store}>
      <AppRouter />
    </Provider>
  );
  ReactDOM.render(jsx, document.getElementById('app'))
  ```
- We're going to use ```connect``` in all of our individual components that need to either **dispatch actions** or **read from the store** 

## Reading from the Store
- What we get back from the API is a function, so we need to call it with the regular component
```const ConnectedExpenseList = connect()(ExpenseList)```
  ```javascript
  import React from 'react'
  import { connect } from 'react-redux'

  const ExpenseList = (props) => (
    <div>
      <h1>Expense List</h1>
      {props.name}<br />             {/* //output: Rocio */}
      {props.expenses.length} <br /> {/* //output: 2 */}
      {props.filters.text}           {/* //output: bill */}
    </div>
  )

  //new const for HOC 
  //what we get back from the API is a function, so we need to call it with the regular component
  const ConnectedExpenseList = connect((state) => {
    //function as parameter, store state get passed in as 1st argument
    //what information from the state we want to access? usually a subset of the state
    //we just need to return an object - any key-value pairs we want, usually from the state but any pair is possible
    return {
      name: 'Rocio', //not too useful
      expenses: state.expenses,
      filters: state.filters
    }

  })(ExpenseList)

  //export the connectedList instead the list
  export default ConnectedExpenseList
  ```
- It's not a common pattern to create a separate variable and then export it by default. It's more common to just export the unnamed call
  ```javascript
  export default connect(state => {
    return {
      name: "Rocio",
      expenses: state.expenses,
      filters: state.filters
    };
  })(ExpenseList)
  ```
- It's also a very common practice to take the function inside ```connect``` and break it out into its own variable usually named ```mapStateToProps```
  ```javascript
  const mapStateToProps = (state) => {
    return {
      name: "Rocio",
      expenses: state.expenses,
      filters: state.filters
    }
  }
  export default connect(mapStateToProps)(ExpenseList)
  ```
- Further simplification by implicitly returning the object
  ```javascript
  const mapStateToProps = (state) => ({
    name: "Rocio",
    expenses: state.expenses,
    filters: state.filters
  })
  ```
- As the store changes ```mapStateToProps``` automatically going to rerun getting the fresh values in the component
  ```javascript
  store.dispatch(setTextFilter('bill'))

  setTimeout(() => {
    store.dispatch(setTextFilter("rent"));
  }, 3000)
  ```
- When you connect a component to the redux store it's reactive which means that as the store changes your component is going to get re-rendered with those new values
- The connected component doesn't need to worry about using ```store.subscribe``` or ```store.getState``` it doesn't have to use any component state to manage that data, instead all of that is done for us by react-redux. All we have to do is define how we want to render things. This is called **Presentational Component Pattern**
- Expenses rendering
  ```javascript
  //ExpenseList.js
  import React from 'react'
  import { connect } from 'react-redux'
  import ExpenseListItem from './ExpenseListItem'
  import selectExpenses from '../selectors/expenses'

  const ExpenseList = (props) => (
    <div>
      <h1>Expense List</h1>
      {
        props.expenses.map((expense) => 
        <ExpenseListItem 
          key={expense.id}
          {...expense}
        />)
      }
    </div>
  )

  const mapStateToProps = state => ({
    expenses: selectExpenses(state.expenses, state.filters)
  });

  export default connect(mapStateToProps)(ExpenseList)

  //ExpenseListItem
  import React from 'react';

  const ExpenseListItem = ( { description, amount, createdAt}) => (
    <div>
      <h3>{description}</h3>
      <p>{amount} - {createdAt}</p>
    </div>
  );

  export default ExpenseListItem
  ```

## Writing to the Store
- Dispatch actions to change data in the store
- ExpenseListFilters is rendered in ExpenseDashboardPage
  ```javascript
  //ExpenseListFilters.js
  import React from 'react'
  import { connect } from 'react-redux'
  import { setTextFilter } from '../actions/filters'

  const ExpenseListFilters = (props) => (
    <div>
      <input type="text" 
        //input renders whatever is in the store
        value={props.filters.text}
        onChange={(event) => {
          //dispatch an action to change filters.text value
          props.dispatch(setTextFilter(event.target.value))
        }}  
      />
    </div>
  )

  const mapStateToProps = (state) => ({
    filters: state.filters
  })
  export default connect(mapStateToProps)(ExpenseListFilters)
  ```
- Remove Expense button - write to the store
  ```javascript
  import React from 'react';
  import { connect } from 'react-redux'
  import { removeExpense } from '../actions/expenses'

  const ExpenseListItem = ( { dispatch, id, description, amount, createdAt }) => (
    <div>
      <h3>{description}</h3>
      <p>{amount} - {createdAt}</p>
      <button onClick={() => {
        dispatch(removeExpense({id}))
      }}>Remove</button>
    </div>
  );

  //We don't want to gather anything from the store 
  //but we want to dispatch an action so it has to be connected
  export default connect()(ExpenseListItem)
  ```
- **Controlled input** are inputs such as ```<input>``` or ```<select>``` where the value is controlled by javascript. The alternative would be a form field. We're going to be sticking with the controlled inputs when we can. It gives us a lot more control  
  ```javascript
  const ExpenseListFilters = (props) => (
    <div>
      <input 
        type="text" 
        //input renders whatever is in the store
        value={props.filters.text}
        onChange={(event) => {
          //dispatch an action to change filters.text value
          props.dispatch(setTextFilter(event.target.value))
        }}  
      />
      <select 
        value={props.filters.sortBy}
        onChange={(event) => {
          if (event.target.value === 'date') {
            props.dispatch(sortByDate())
          } else if (event.target.value === 'amount') {
            props.dispatch(sortByAmount());
          }
      }}>
        <option value="date">Date</option>
        <option value="amount">Amount</option>
      </select>
    </div>
  )
  ```

## Handling the Form
- The idea here is to keep track of the changes to the form input in local state and then when clicking submit change the store

### input and textarea
- There are two ways to manage the onChange event
  - Create a constant first - input
    ```javascript
    //file ExpenseForm.js
    import React from 'react'

    export default class ExpenseForm extends React.Component {
      state = {
        description: '',
      }
      onDescriptionChange = (event) => {
        //constant to be used in the callback
        const description = event.target.value
        this.setState(() => ( { description })) //returns implicit object
      }
      render() {
        return (
          <div>
            <form>
              <input
                type="text"
                placeholder="Description"
                autoFocus
                value={this.state.description}
                onChange={this.onDescriptionChange}
              />
              <button>Add Expense</button>
            </form>
          </div>
        )
      }
    }
    ```
  - Using persist to being able to use the event inside the callback - textarea
    ```javascript
    import React from 'react'

    export default class ExpenseForm extends React.Component {
      state = {
        note: ''
      }
      onNoteChange = (event) => {
        //if we want to use event in the callback we need to persist the event
        event.persist()
        this.setState(() => ( { note: event.target.value }))
      }
      render() {
        return (
          <div>
            <form>
              <textarea
                placeholder="Add a note for your expense (optional)"
                value={this.state.note}
                onChange={this.onNoteChange}
              >
              </textarea>
              <button>Add Expense</button>
            </form>
          </div>
        )
      }
    }
    ```
  
### amount field
- The ```number``` input type restrict the input however accepts all decimals we want. It's better to switch it to text and manage the validation manually with a regular expression
- https://regex101.com is a good site to check your regEx
  ```javascript
  import React from 'react'

  export default class ExpenseForm extends React.Component {
    state = {
      amount: ''
    }
    onAmountChange = (event) => {
      const amount = event.target.value
      //regEx: 
      //^ starts with 
      // \d digit
      //* zero ar as many as you want
      //() optional
      //\. escapes the dot
      //{0, 2} min 0 max 2
      //? 0 or 1
      //$ ends with
      //amount.match(/^\d*(\.\d{0,2})?$/))
      //{1,} one or as many as you want
      //!amount allows to delete the value
      if (!amount || amount.match(/^\d{1,}(\.\d{0,2})?$/)) {
        this.setState(() => ( { amount }))
      }  
    }
    render() {
      return (
        <div>
          <form>
            <input 
              type="text"
              placeholder="Amount"
              value={this.state.amount}
              onChange={this.onAmountChange}
            />
            <button>Add Expense</button>
          </form>
        </div>
      )
    }
  }
  ```

### createdAt field
- We're going to setup a date picker 
- We need two important libraries **moment.js** which is a time library that makes easy to manipulate and format time (https://momentjs.com)
- We'll also use airbnb/react-dates which is a component we will use to allow the user to pick a date (https://github.com/airbnb/react-dates & http://airbnb.io/react-dates)
  ```javascript
  yarn add moment react-dates (I did not install react-addons-compare, it was in my version)
  ```
- react-dates needs a ```moment``` object to get initialized. Also when the user picks a date we get back a moment object
  ```javascript
  import moment from 'moment'

  const now = moment()
  console.log(now)
  console.log(now.format('MMM Do, YYYY'))
  ```
- Using airbnb SingleDatePicker
  ```javascript
  import React from 'react'
  //react-dates uses moment as objects to initialize and return
  import moment from 'moment'
  import { SingleDatePicker } from 'react-dates'
  //this one is required in newer versions than the video
  import 'react-dates/initialize';
  import 'react-dates/lib/css/_datepicker.css'

  export default class ExpenseForm extends React.Component {
    state = {
      createdAt: moment(),
      //if we want it to be displayed when the page loads
      calendarFocused: false
    }
    //gets the selected date passed in
    onDateChange = (createdAt) => {
      //this if avoids to delete the date
      if (createdAt) {
        this.setState(() => ({ createdAt }))
      }
    }
    onFocusChange = ( {focused}) => {
      this.setState(() => ({ calendarFocused: focused }))
    }
    render() {
      return (
        <div>
          <form>
            <SingleDatePicker 
              //initial date
              date={this.state.createdAt}
              onDateChange={this.onDateChange}
              focused={this.state.calendarFocused}
              onFocusChange={this.onFocusChange}
              //by default shows two months at once, here we change to 1
              numberOfMonths={1}
              //to allow selecting dates in the past, basically no date is outside range
              isOutsideRange={ () => false }
              //required in newer version
              id="createdAt"
            />
            <button>Add Expense</button>
          </form>
        </div>
      )
    }
  }
  ```

### Validate the form
- Add error to the state
- Add onSubmit handler
- Prevent the form from regular submit
  ```javascript
  export default class ExpenseForm extends React.Component {
    state = {
      ...
      error: ''
    }
    ....
    onSubmit = (event) => {
      event.preventDefault()
      //if description or amount are empty set the error message
      if(!this.state.description || !this.state.amount) {
        this.setState(() => ({ error: 'Please provide description and amount.'}))
      } else {
        // if both values are provided clear the error
        this.setState(() => ({ error: '' }))
        console.log('submitted')
      }
    }
    render() {
      return (
        <div>
          {/* render error message if there is one */}
          {this.state.error && <p>{this.state.error}</p>}
          <form onSubmit={this.onSubmit}>
            ...
            <button>Add Expense</button>
          </form>
        </div>
      )
    }
  }
  ```
- We don't want to dispatch the action to create an expense from ```ExpenseForm.js``` because we want the Form component to be reused for editing an expense. So the action we dispatch is going to be different for add and edit
- What we're going to do is just pass the data up and then determine what to do with the data when the user submits the form dynamically. So we define a function in AddExpensePage that dispatch the action but the actual call to the function is done in the children ExpenseForm
  ```javascript
  //AddExpensePage.js
  import React from 'react'
  import ExpenseForm from './ExpenseForm'
  import { connect } from 'react-redux' // we need connect to be able to dispatch actions
  import { addExpense } from '../actions/expenses'

  const AddExpensePage = (props) => (
    <div>
      <h1>Add Expense</h1>
      <ExpenseForm 
        //onSubmit gets passed to the Form as prop so that it can be executed from there
        //expense is an object that gets filled in inside the children ExpenseForm
        onSubmit={(expense) => {
          //dispatch the action addExpense
          props.dispatch(addExpense(expense))
          //redirect to the dashboard page
          //history is part of the properties because the router
          props.history.push('/')
        }}
      />
    </div>
  );

  //don't need anything from the store but we want to connect this component to dispatch addExpense action
  export default connect()(AddExpensePage)
  ```
  ```javascript
  //ExpenseForm.js
  onSubmit = (event) => {
    event.preventDefault()
    if(!this.state.description || !this.state.amount) {
      this.setState(() => ({ error: 'Please provide description and amount.'}))
    } else {
      this.setState(() => ({ error: '' }))
      //onSubmit is a method defined in the parent AddExpensePage, we call it from the child  
      this.props.onSubmit({
        description: this.state.description,
        amount: parseFloat(this.state.amount, 10) * 100, //we're working on cents
        createdAt: this.state.createdAt.valueOf(), //get milliseconds out of the moment object
        note: this.state.note
      })
    }
  }
  ```

### Edit Expense
- Reuses the ExpenseForm component to pre-populate the expense values
- To have access to props in the initial state need to refactor and introduce the constructor in ExpenseForm
- In EditExpense we're passing down one more prop with the expense
  ```javascript
  import React from 'react'
  import { connect } from 'react-redux'
  import ExpenseForm from './ExpenseForm'
  import { editExpense, removeExpense } from '../actions/expenses'

  const EditExpensePage = (props) => {
    return (
      <div>
        <ExpenseForm 
          //passing the expense to the form
          expense={props.expense}
          onSubmit={(expense) => {
            props.dispatch(editExpense(props.expense.id, expense))
            props.history.push('/')
          }}
        />
        <button onClick={() => {
          props.dispatch(removeExpense({ id: props.expense.id }))
          props.history.push('/')
        }}>Remove</button>
      </div>
    );
  }

  const mapStateToProps = (state, props) => {
    return {
      expense: state.expenses.find((expense) => expense.id === props.match.params.id)
    }
  }

  export default connect(mapStateToProps)(EditExpensePage)
  ```
- In the form if expense is coming we pre-populate the fields otherwise use the default initial values
  ```javascript
  import React from 'react'
  import moment from 'moment'
  import { SingleDatePicker } from 'react-dates'
  import 'react-dates/lib/css/_datepicker.css'

  export default class ExpenseForm extends React.Component {
    //change to have a constructor so that we can access props
    //if expense is coming as a prop means EditPage is calling this component not AddPage
    //set state with current values for edit or default initial values for add
    constructor(props) {
      super(props)
      this.state = {
        description: props.expense ? props.expense.description : '',
        note: props.expense ? props.expense.note : '',
        amount: props.expense ? (props.expense.amount / 100).toString() : '',
        createdAt: props.expense ? moment(props.expense.createdAt) : moment(),
        calendarFocused: false,
        error: ''
      }
    }
    
    onDescriptionChange = (event) => {
      const description = event.target.value
      this.setState(() => ( { description })) //returns implicit object
    }
    onNoteChange = (event) => {
      //if we want to use event in the callback we need to persist the event
      event.persist()
      this.setState(() => ( { note: event.target.value }))
    }
    onAmountChange = (event) => {
      const amount = event.target.value
      if (!amount || amount.match(/^\d{1,}(\.\d{0,2})?$/)) {
        this.setState(() => ( { amount }))
      }   
    }
    onDateChange = (createdAt) => {
      //this if avoids to delete the date
      if (createdAt) {
        this.setState(() => ({ createdAt }))
      }
    }
    onFocusChange = ( {focused}) => {
      this.setState(() => ({ calendarFocused: focused }))
    }
    onSubmit = (event) => {
      event.preventDefault()

      if(!this.state.description || !this.state.amount) {
        this.setState(() => ({ error: 'Please provide description and amount.'}))
      } else {
        this.setState(() => ({ error: '' }))
        //onSubmit is a method defined in the parent AddExpensePage or EditExpensePage, we call it from the child  
        this.props.onSubmit({
          description: this.state.description,
          amount: parseFloat(this.state.amount, 10) * 100, //we're working on cents
          createdAt: this.state.createdAt.valueOf(),
          note: this.state.note
        })
      }
    }
    render() {
      return (
        <div>
          {this.state.error && <p>{this.state.error}</p>}
          <form onSubmit={this.onSubmit}>
            <input
              type="text"
              placeholder="Description"
              autoFocus
              value={this.state.description}
              onChange={this.onDescriptionChange}
            />
            <input 
              type="text"
              placeholder="Amount"
              value={this.state.amount}
              onChange={this.onAmountChange}
            />
            <SingleDatePicker 
              date={this.state.createdAt}
              onDateChange={this.onDateChange}
              focused={this.state.calendarFocused}
              onFocusChange={this.onFocusChange}
              numberOfMonths={1}
              isOutsideRange={ () => false }
              id="createdAt"
            />
            <textarea
              placeholder="Add a note for your expense (optional)"
              value={this.state.note}
              onChange={this.onNoteChange}
            >
            </textarea>
            <button>Add Expense</button>
          </form>
        </div>
      )
    }
  }
  ```

## Redux Developer Tools
- Install extension in chrome
- Add configuration to the store
- https://github.com/zalmoxisus/redux-devtools-extension
  ```javascript
  // Store creation
  export default () => {
    const store = createStore(
      combineReducers({
        expenses: expensesReducer,
        filters: filtersReducer
      }),
      //this info is added for chrome extension Redux DevTools
      window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
    )
    return store
  }
  ```
- Possible to check actions, state, differences, dispatch actions and watch how the store changes

## Filtering by dates
- Change the filters file
  ```javascript
  //src/reducers/filters.js
  import moment from 'moment'

  const filtersReducerDefaultState = {
    text: '',
    sortBy: 'date',
    //by default we'll pick the 1st and last day of the current month as the range
    startDate: moment().startOf('month'),
    endDate: moment().endOf('month')
  }

  export default (state = filtersReducerDefaultState, action) => { .... }
  ```
- Change ExpenseListFilters to class component because we need to track state. When we convert we need to change ```props``` for ```this.props```
- Include the DateRangePicker component
  ```javascript
  import React from 'react'
  import { connect } from 'react-redux'
  import { DateRangePicker } from 'react-dates'
  import { setTextFilter, sortByDate, sortByAmount, setStartDate, setEndDate } from '../actions/filters'

  class ExpenseListFilters extends React.Component {
    state = {
      calendarFocused: null 
    }
    //this function gets called by the react-dates library
    //it's gonna get called with an object and in that object we'll have startDate and endDate
    //we can destructure or have a variable and pull the values off
    onDatesChange = ({ startDate, endDate }) => {
      this.props.dispatch(setStartDate(startDate))
      this.props.dispatch(setEndDate(endDate))
    }
    onFocusChange = (calendarFocused) => {
      this.setState(() => ({ calendarFocused }))
    }
    render() {
      return (
        <div>
          {...}
          <DateRangePicker
            startDate={this.props.filters.startDate}
            startDateId="startDate"
            endDate={this.props.filters.endDate}
            endDateId="endDate"
            onDatesChange={this.onDatesChange}
            focusedInput={this.state.calendarFocused} //toggle visualization of the calendar
            onFocusChange={this.onFocusChange}
            showClearDates={true} //shows and X to clear the range
            numberOfMonths={1} //one month at a time
            isOutsideRange={() => false} //allows to pick dates in the past
          />
        </div>
      )
    }
  }
  const mapStateToProps = (state) => ({
    filters: state.filters
  })
  export default connect(mapStateToProps)(ExpenseListFilters)
  ```
- Fix the application of the filters, before we were using numbers, now we are using moment instances
  ```javascript
  //file: src/selectors/expenses
  import moment from 'moment'
  // Get visible expenses

  export default (expenses, { text, sortBy, startDate, endDate }) => {
    return expenses.filter((expense) => {
      const createdAtMoment = moment(expense.createdAt)
      const startDateMatch = startDate ? startDate.isSameOrBefore(createdAtMoment, 'day') : true
      const endDateMatch = endDate ? endDate.isSameOrAfter(createdAtMoment, 'day') : true
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

# Yarn aliases
- ```test``` and ```start``` in yarn have aliases or short hands because they're very popular
- yarn run start and yarn start are the same
- yarn run test and yarn test are the same
 
# Podcasts
https://github.com/rShetty/awesome-podcasts#functional-programming

# Testing
- Jest (from Facebook) integrates well with react and as such the best to work with in React Apps. There are other testing frameworks, each one fulfills their own purposes jasmine, moka (node), karma (angular)
- yarn add jest
- package.json => new script 
  ```javascript
  "scripts": {
    "serve": "live-server public/",
    "build": "webpack",
    "dev-server": "webpack-dev-server",
    "test": "jest"
  },
  ``` 
- https://github.com/facebook/jest /  https://jestjs.io/docs/en/api   https://jestjs.io
- extension of files is fileName.test.js - jest will watch for this extension when looking for tests to run
- Inside the testing files we get access to a set of global variables that jest provides to us allowing us to construct test cases eg. ```test``` which receives two arguments. 1st is a string describing what the test should do and 2nd the code to run for the test case. So, usually a string and an arrow function.
  ```javascript
  const add = (a, b) => a + b + 1

  test ('should add two numbers', () => {
    const result = add (4, 3)

    //manual assertion, tedious and error prone
    if (result !== 7) {
      throw new Error(`You added 4 and 3. The result was ${result}. Expected 7`)
    }
  })
  ```
- jest gives us an assertion library. ```expect``` is another global that jest provides
  ```javascript
  const add = (a, b) => a + b

  test ('should add two numbers', () => {
    const result = add (4, 3)
    expect(result).toBe(7)
  })
  ```
- test files are run through babel so we can use ES6 and ES7 features
- We can run jest in watch mode, when a test changes or one of the things the file imports change, we're going to rerun the test suite
  - change the script in package.json. Although sometimes we don't want to watch changes just run the tests.
  ```javascript
  "test": "jest --watch"
  ```
  - type a different command in the terminal
    ```javascript
    yarn run test -- --watch //the first -- means the next are parameter passed down to the script not to the yarn command - video version
    ```
  - with my version the jest the command above didn't work. Used:
  ```javascript
  yarn run test --watchAll
  ```
- Challenge
  ```javascript
  const generateGreeting = (name = 'Anonymous') => (`Hello ${name}!`)

  test ('should generate greeting from name', () => {
    const greeting = generateGreeting('Rocio')
    expect(greeting).toBe('Hello Rocio!')
  })

  test ('should generate greeting for no name', () => {
    const greeting = generateGreeting()
    expect(greeting).toBe('Hello Anonymous!')
  })
  ```

## Testing Action Generators
- toBe is not suitable to compare objects because it uses === to compare. Even in the console {} === {} and [] === [] will return in false. We use toEqual instead, it goes though the array or object and asserts that all properties are the same
- We can assert the type of a value if we don't know what the exact value is gonna be. Like in the uuid generator
  ```javascript
  //expenses.test.js
  import { addExpense, removeExpense, editExpense } from '../../actions/expenses'

  //amount in this file is a number representing cents
  //createdAt in this file is a number
  test('should setup remove expense action object', () => {
    const action = removeExpense({ id: '123AbC'})
    expect(action).toEqual({
      type: 'REMOVE_EXPENSE',
      id: '123AbC'
    })
  })

  test('should setup edit expense action object', () => {
    const action = editExpense('123AbC', { description: 'New description', amount: 2550})
    expect(action).toEqual({
      type: 'EDIT_EXPENSE',
      id: '123AbC',
      updates: {
        description: 'New description',
        amount: 2550
      }
    })
  })

  test('should setup add expense action object with provided values', () => {
    const expenseData = {
      description: 'Rent',
      amount: 109500,
      createdAt: 1000,
      note: 'This was November Rent'
    } 
    const action = addExpense(expenseData)
    expect(action).toEqual({
      type: 'ADD_EXPENSE',
      expense: {
        ...expenseData, //spread out expenseData
        id: expect.any(String) //id changes every time
      }
    })
  })

  test('should setup add expense action object with default values', () => {
    const action = addExpense()
    expect(action).toEqual({
      type: 'ADD_EXPENSE',
      expense: {
        id: expect.any(String), //id changes every time
        description: '',
        note: '',
        amount: 0,
        createdAt: 0
      }
    })
  })
  ```

## Testing Selector
  ```javascript
  //file src/tests/selectors/expenses.test.js
  import selectExpenses from '../../selectors/expenses'
  import moment from 'moment'

  //createdAt in the expenses and filters objects are timestamps / numbers
  const expenses = [{
    id: '1',
    description: 'Gum',
    note: '',
    amount: 195,
    createdAt: 0 //unix epoch time - 1st January 1970
  }, {
    id: '2',
    description: 'Rent',
    note: '',
    amount: 109500,
    createdAt: moment(0).subtract(4, 'days').valueOf() //4 days in the past
  },
  {
    id: '3',
    description: 'Credit card',
    note: '',
    amount: 4500,
    createdAt: moment(0).add(4, 'days').valueOf() //4 days in the future
  }]

  test('should filter by text value', () => {
    const filters = {
      text: 'e',
      sortBy: 'date',
      startDate: undefined,
      endDate: undefined
    }
    const result = selectExpenses(expenses, filters)
    expect(result.length).toBe(2)
    //credit card comes first because by default we sort by date, newer first
    expect(result).toEqual([ expenses[2], expenses[1] ])
  })

  test('should filter by start date', () => {
    const filters = {
      text: '',
      sortBy: 'date',
      startDate: moment(0),
      endDate: undefined
    }
    const result = selectExpenses(expenses, filters)
    expect(result.length).toBe(2)
    //the expense in the past gets filtered out
    expect(result).toEqual([ expenses[2], expenses[0] ])
  })

  test('should filter by end date', () => {
    const filters = {
      text: '',
      sortBy: 'date',
      startDate: undefined,
      endDate: moment(0).add(2, 'days')
    }
    const result = selectExpenses(expenses, filters)
    expect(result.length).toBe(2)
    //the expenses 2 days after unix epoch time get filtered out
    expect(result).toEqual([ expenses[0], expenses[1] ])
  })

  test('should sort by date', () => {
    const filters = {
      text: '',
      sortBy: 'date',
      startDate: undefined,
      endDate: undefined
    }
    const result = selectExpenses(expenses, filters)
    expect(result.length).toBe(3)
    //newer first
    expect(result).toEqual([ expenses[2], expenses[0], expenses[1] ])
  })

  test('should sort by amount', () => {
    const filters = {
      text: '',
      sortBy: 'amount',
      startDate: undefined,
      endDate: undefined
    }
    const result = selectExpenses(expenses, filters)
    expect(result.length).toBe(3)
    expect(result).toEqual([ expenses[1], expenses[2], expenses[0] ])
  })
  ```
## Testing reducers

### Filters
  ```javascript
  import moment from 'moment'
  import filtersReducer from '../../reducers/filters'

  test('should setup default filter values' , () => {
    //there's an internal action @@INIT that gets called when initializing
    //undefined existing state and action type @@INIT
    const state = filtersReducer(undefined, { type: '@@INIT'})
    expect(state).toEqual({
        text: '',
        sortBy: 'date',
        startDate: moment().startOf('month'),
        endDate: moment().endOf('month')
    })
  })

  test('should set sortBy to amount', () => {
    //by default it's date, we're verifying the change
    const state = filtersReducer(undefined, { type: 'SORT_BY_AMOUNT'})
    expect(state.sortBy).toBe('amount')
  })

  test('should set sortBy to date', () => {
    const currentState = {
      text: '',
      sortBy: 'amount', //make sure it starts with amount so that we can watch the change
      startDate: undefined,
      endDate: undefined
    }
    const action = {
      type: 'SORT_BY_DATE'
    }
    const state = filtersReducer(currentState, action)
    expect(state.sortBy).toBe('date')
  })

  test('should set text filter', () => {
    //it's good idea to create a const for this as we have to use it in two places
    const text = 'This is my filter' 
    const state = filtersReducer(undefined, {  //second parameter is the action
      type: 'SET_TEXT_FILTER', 
      text
    })
    expect(state.text).toBe(text)
  })

  test('should set start date filter', () => {
    //the default value startOf month was already tested so we don't need to repeat
    //the purpose of the test is to check the filter is set
    const startDate = moment()
    const state = filtersReducer(undefined, { 
      type: 'SET_START_DATE', 
      startDate
    })
    expect(state.startDate).toEqual(startDate)
  })

  test('should set end date filter', () => {
    //the default value endOf month was already tested so we don't need to repeat
    //the purpose of the test is to check the filter is set
    const endDate = moment()
    const state = filtersReducer(undefined, { 
      type: 'SET_END_DATE', 
      endDate
    })
    expect(state.endDate).toEqual(endDate)
  })
  ```

### Expenses
  ```javascript
  import expensesReducer from '../../reducers/expenses'
  import expenses from '../fixtures/expenses'
  import moment from 'moment'

  test('should set default state', () => {
    const state = expensesReducer(undefined, {type: '@@INIT'})
    expect(state).toEqual([])
  })

  test('should remove expense by id', () => {
    const action = {
      type: 'REMOVE_EXPENSE',
      id: expenses[1].id
    }
    const state = expensesReducer(expenses, action)
    expect(state.length).toBe(2)
    expect(state).toEqual([ expenses[0], expenses[2] ])
  })

  test('should not remove expenses if id not found', () => {
    const action = {
      type: 'REMOVE_EXPENSE',
      id: '-1'
    }
    const state = expensesReducer(expenses, action)
    expect(state.length).toBe(3)
    expect(state).toEqual(expenses)
  })

  test('should add an expense', () => {
    const expense = {
      id: '100',
      description: 'desc',
      note: '',
      amount: 12500,
      createdAt: moment() //today
    }
    const action = {
      type: 'ADD_EXPENSE',
      expense
    }
    const state = expensesReducer(expenses, action)
    expect(state.length).toBe(4)
    expect(state).toEqual([...expenses, expense])
  })

  test('should edit an expense', () => {
    const updates = {
      description: 'New description',
      note: 'New note',
      amount: 25000,
      createdAt: moment().subtract(1, 'day') //yesterday
    }
    const action = {
      type: 'EDIT_EXPENSE',
      id: expenses[0].id,
      updates
    }
    const state = expensesReducer(expenses, action)
    expect(state.length).toBe(3)
    expect(state[0]).toEqual({
      id: '1',
      ...updates
    })
  })

  test('should edit an expense 1 property', () => {
    const amount = 25000
    const action = {
      type: 'EDIT_EXPENSE',
      id: expenses[0].id,
      updates: {
        amount
      }
    }
    const state = expensesReducer(expenses, action)
    expect(state.length).toBe(3)
    expect(state[0].amount).toBe(amount)
  })

  test('should not edit expense if expense not found', () => {
    const updates = {
      description: 'New description',
      note: 'New note',
      amount: 25000,
      createdAt: moment().subtract(1, 'day') //yesterday
    }
    const action = {
      type: 'EDIT_EXPENSE',
      id: '-1',
      updates
    }
    const state = expensesReducer(expenses, action)
    expect(state.length).toBe(3)
    expect(state).toEqual(expenses)
  })

  ```

# Fixtures
- In the test world, a fixture is just your baseline test data or dummy data 
  ```javascript
  //file: src/tests/fixtures/expenses.js
  import moment from 'moment'

  export default [{
    id: '1',
    description: 'Gum',
    note: '',
    amount: 195,
    createdAt: 0 //unix epoch time - 1st January 1970
  }, {
    id: '2',
    description: 'Rent',
    note: '',
    amount: 109500,
    createdAt: moment(0).subtract(4, 'days').valueOf() //4 days in the past
  },
  {
    id: '3',
    description: 'Credit card',
    note: '',
    amount: 4500,
    createdAt: moment(0).add(4, 'days').valueOf() //4 days in the future
  }]
  ```
- Then we import the fixture wherever we need and use it as per usual
  ```javascript
  import expenses from '../fixtures/expenses'

  console.log(expenses) //print the whole array
  ```