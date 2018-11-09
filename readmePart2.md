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
