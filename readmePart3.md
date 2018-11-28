> markdown: https://guides.github.com/features/mastering-markdown/  
> Preview: Chrome extension and QuickLook / space key on the file 
> TOC failing if EOL enabled https://github.com/AlanWalk/markdown-toc/issues/65

React Course - Udemy - Andrew Mead - Part III
===============================================
<!-- TOC -->

- [Deploying your Apps](#deploying-your-apps)
  - [Git](#git)
  - [Github / SSH keys](#github--ssh-keys)
  - [Production Webpack](#production-webpack)
  - [Creating separate CSS files](#creating-separate-css-files)
  - [Production server with express](#production-server-with-express)
  - [Heroku](#heroku)
- [New feature workflow](#new-feature-workflow)
  - [Adding Total selector](#adding-total-selector)
  - [New component ExpensesSummary](#new-component-expensessummary)
- [Firebase](#firebase)
  - [Connecting to the database](#connecting-to-the-database)
  - [Writing to the database](#writing-to-the-database)
  - [Promises](#promises)
  - [Promises with Firebase](#promises-with-firebase)
    - [Add data](#add-data)
    - [Remove data](#remove-data)
    - [Update data](#update-data)
    - [Fetch data](#fetch-data)
      - [Fetch data a single time getting the object back](#fetch-data-a-single-time-getting-the-object-back)
      - [Fetch data and subscribe to changes](#fetch-data-and-subscribe-to-changes)
    - [Arrays in Firebase](#arrays-in-firebase)
- [Firebase with Redux](#firebase-with-redux)
  - [Asynchronous Redux Actions](#asynchronous-redux-actions)
  - [Testing Async Redux actions](#testing-async-redux-actions)
    - [Check dispatched action](#check-dispatched-action)
    - [Checking the database was updated](#checking-the-database-was-updated)
  - [Creating a separate database](#creating-a-separate-database)
  - [Fetching expenses](#fetching-expenses)

<!-- /TOC -->

# Deploying your Apps

## Git
- Install git
- ```git --version```
- Initialize repository ```git init``` from the root of the project. It creates the .git directory
- ```git status``` high level overview of the repo at the current point in time
- Git Pro book https://git-scm.com/book/en/v2
- Create file ```.gitignore``` in the root of the project and add ```node_modules/``` as we don't want to track that folder
- Move files from the untracked area to the staged changes area ```git add package.json```
- Files in the staged area will be taken by the next commit
- To add all files we can use a shortcut ```git add .``` which will add everything in the current directory and subdirectories. ```git reset``` to undo
- Commit the changes ```git commit -m "[Rocio] initial commit"```
- Check rec commits ```git log```
  ```javascript
  git init - Create a new git repo
  git status - View the changes to your project code
  git add - Add files to staging area
  git commit - Creates a new commit with files from the staging area
  git log - View recent commits

  Loop - status / add / commit / status
  ```

## Github / SSH keys
- Login or Sign up to github
- Create new public repository
- To be able to communicate with github from our machine we need to establish a trusted communication channel by using ssh keys https://help.github.com/articles/connecting-to-github-with-ssh/
- Check for existing keys ```ls -a ~/.ssh```
- Create keys ```ssh-keygen -t rsa -b 4096 -C "your_email@example.com"``` accept defaults
- Check the tool ssh-agent is running ```eval "$(ssh-agent -s)"``` if it's not running it will start it up
- Add the key ```ssh-add -K ~/.ssh/id_rsa```
- Take the public key file and give it to github
- Get the content of the public key
  ```javascript
  pbcopy < ~/.ssh/id_rsa.pub
  # Copies the contents of the id_rsa.pub file to your clipboard
  ```
- In github settings/SSH keys add the key and save
- Test the connection ```ssh -T git@github.com```
- Copy the ssh URL of the repo created in github
- In the terminal execute this to link or associate our local project with the repo ```git remote add origin git@github.com:rossyp44Y/rocio-expensify-app.git```
- git remote -v to see the urls fetch and push
- First time pushing we need to use the -u flag ```push -u origin master```

## Production Webpack
- Create separate script to set up production script build by adding the -p flag. This flag minify our javascript files and also sets the production environment variable for our third-party libraries
  ```javascript
  "build:dev": "webpack",
  "build:prod": "webpack -p",
  ```
- We also set the env production flag, allowing us to customize how webpack.config returns the config object. We switch from exporting an object to exporting a function that returns the object, the difference is that the function can take in parameters such as the environment. If we're  in production we use one type of source map if we're not in production we use a different type of source map. This reduces the total size of the bundle.js but now creates two files one with the bundle and another one with the source maps which will only be loaded if we open dev-tools
  ```javascript
  "scripts": {
    ...
    "build:dev": "webpack",
    "build:prod": "webpack -p --env production",
    ...
  },
  ```
  ```javascript
  //webpack.config.js
  const path = require('path')

  //expose this object to another file - node syntax
  module.exports = (env) => {
    const isProduction = env === 'production'
    return {
      entry: './src/app.js',
      output: {
        path: path.join(__dirname, 'public'), //absolute path on your machine to the public folder
        filename: 'bundle.js'
      },
      module: {
        rules: [{
          loader: 'babel-loader',
          test: /\.js$/,   //files that ends in .js
          exclude: /node_modules/
        }, {
          test: /\.s?css$/,   //files that ends in .css and .scss
          use: [  //an array of loaders
            'style-loader',
            'css-loader',
            'sass-loader'
          ]
        }]
      },
      devtool: isProduction ? 'source-map' : 'cheap-module-eval-source-map',
      devServer: {
        contentBase: path.join(__dirname, 'public') ,
        historyApiFallback: true
      }
    }
  }
  ```
- Run prod build ```yarn run build:prod```
- Size decreased and command takes a bit longer to execute
- Start the server with ```yarn run serve```
- Info: https://webpack.js.org/guides/production/ 
- https://webpack.js.org/configuration/configuration-types/#src/components/Sidebar/Sidebar.jsx


## Creating separate CSS files
- Our css all live inside bundle.js this make the file larger that it should be but it's also doing things it shouldn't
- When the styles are in bundle.js they don't actually get added to the browser until after the javascript runs which takes some time
- We're going to change webpack so it outputs a javascript file that contains just javascript and a separate css file which contains all of our styles. Then we can link the css in index.html 
- Integrate webpack plugin ```extract-text-webpack-plugin```, this will allow us to extract some parts from bundle.js after processing take css and scss files and put them in a separate file
- https://github.com/webpack-contrib/extract-text-webpack-plugin
- ```yarn add extract-text-webpack-plugin```
- Change up webpack.config.js
  ```javascript
  const path = require('path')
  //import the plugin
  const ExtractTextPlugin = require('extract-text-webpack-plugin')

  //expose this object to another file - node syntax
  module.exports = (env) => {
    const isProduction = env === 'production'
    //pass the name of the file styles.css
    const CSSExtract = new ExtractTextPlugin('styles.css')
    return {
      entry: './src/app.js',
      output: {
        path: path.join(__dirname, 'public'), //absolute path on your machine to the public folder
        filename: 'bundle.js'
      },
      module: {
        rules: [{
          loader: 'babel-loader',
          test: /\.js$/,   //files that ends in .js
          exclude: /node_modules/
        }, {
          test: /\.s?css$/,   //files that ends in .css and .scss
          //an array of loaders
          use: CSSExtract.extract({
            use: [
              'css-loader',
              'sass-loader'
            ]
          })
        }]
      },
      plugins: [
        CSSExtract
      ],
      devtool: isProduction ? 'source-map' : 'cheap-module-eval-source-map',
      devServer: {
        contentBase: path.join(__dirname, 'public') ,
        historyApiFallback: true
      }
    }
  }
  ```
- Run the prod build ```yarn run build:prod```
- Add link tag into index.html before closing head
  ```javascript
  <link rel="stylesheet" type="text/css" href="/styles.css"/>
  ```
- Start live server ```yarn run serve```
- Also fix the source maps for css by changing
  ```javascript
  devtool: isProduction ? 'source-map' : 'inline-source-map',
  ```
  ```javascript
  {
    test: /\.s?css$/,   //files that ends in .css and .scss
    //an array of loaders
    use: CSSExtract.extract({
      use: [
        {
          loader: 'css-loader',
          options: {
            sourceMap: true
          }
        },
        {
          loader: 'sass-loader',
          options: {
            sourceMap: true
          }
        }
      ]
    })
  }
  ```
- yarn run dev-server

## Production server with express
- Servers such as live-server and webpack-dev-server are great for development but they're not suitable for production
- yarn add express
- node file to be run in command line with ```node server/server.js```
- http://expressjs.com/en/4x/api.html#app
  ```javascript
  //server/server.js
  const path = require('path')
  const express = require('express')
  //create an express application
  const app = express()
  const publicPath = path.join(__dirname, '..', 'public')
  
  //use the public directory to serve up all static assets
  app.use(express.static(publicPath))

  //match all unmatched routes
  //if what is requested is not in public folder, return index.html
  app.get('*', (req, res) => {
    res.sendFile(path.join(publicPath, 'index.html'))
  })
  //start up in port 3000
  app.listen(3000, () => {
    console.log('Server is up!')
  })
  ```

## Heroku
- Heroku is an application deployment platform similar to AWS Beanstalk or Digital Ocean
- https://www.heroku.com
- Heroku CLI https://devcenter.heroku.com/articles/heroku-cli
- Download, install and check version: ```heroku --version```
- Authenticate: heroku login
- Create App:  ```heroku create rocio-react-expensify``` If no name is specified, a random one will be used. This command sets up the new application and also adds a new git remote to the local repository
- ```git remote``` will show up two remotes: origin and heroku. ```git remote -v``` will verbose both fetch and push urls for both remotes
- To deploy we're gonna push our code to the heroku remote, heroku will get that code and deploy
- We need to make some changes specific for heroku
- When Heroku starts up our application it's going to try to run the start script in package.json in there we need to specify to run our express server
  ```javascript
    "scripts": {
      ...
      "start": "node server/server.js"
    },
  ```
- Heroku will provide a dynamic port value in an environment variable so we can't hardcode 3000, that's ok for development but not for production
  ```javascript
  const path = require('path')
  const express = require('express')
  const app = express()
  const publicPath = path.join(__dirname, '..', 'public')
  //PORT in the env variable heroku will set up for us
  //if it doesn't exist that means we're in local an we use 3000
  const port = process.env.PORT || 3000

  app.use(express.static(publicPath))

  //match all unmatched routes
  app.get('*', (req, res) => {
    res.sendFile(path.join(publicPath, 'index.html'))
  })
  app.listen(port, () => {
    console.log('Server is up!')
  })
  ```
- We need to teach heroku how to run webpack. There are a couple of Heroku script names we can use to specify tasks. heroku-prebuild which we're not using and heroku-postbuild which will run after installing all dependencies. We also need to add to gitignore the four files that are generated when we run build:prod locally.
  ```javascript
  //package.json add script
  "heroku-postbuild": "yarn run build:prod"

  //.gitignore
  node_modules/
  public/bundle.js
  public/bundle.js.map
  public/styles.css
  public/styles.css.map
  ``` 
- Commit and push changes
  ```javascript
  git add .
  git commit -m "setup production build and server"
  git push //to github
  git push heroku master //remote branch - takes some time
  heroku open //or we can copy the url and paste it in the browser
  https://rocio-react-expensify.herokuapp.com
  heroku logs
  ```
- Another improvement is to move dependencies used only for development purposes such as enzyme, live-server and webpack-dev-server to a different section in package.json called ```devDependencies``` so that they are not installed in heroku 
- When installing a dependency from the terminal use ```yarn add chalk --dev```.
  ```javascript
  //moved to devDependencies
  "devDependencies": {
    "enzyme": "^3.7.0",
    "enzyme-adapter-react-16": "^1.7.0",
    "enzyme-to-json": "^3.3.4",
    "jest": "^23.6.0",
    "react-test-renderer": "^16.6.3",
    "webpack-dev-server": "2.5.1"
  }
  //removed because we don't need them anymore
  dependency: "live-server": "^1.2.0",
  script: "serve": "live-server public/",
  ```
- Now, how to install one or another? ```yarn install --production``` will install everything under ```dependencies``` and leave out the ```devDependencies``` This is the one used by heroku. ```yarn install``` will continue installing both, regular dependencies and dev dependencies
- Next improvement is to create a dist folder inside public to put the four compiled files webpack generates instead of having the four files along with other files such as index.html. This will also change ```.gitignore``` as we can now ignore the whole directory
- Change index.html to refer to the new structure
  ```javascript
  <link rel="stylesheet" type="text/css" href="/dist/styles.css"/>
  <script src="dist/bundle.js"></script>
  ```
- Change webpack.config.js to dump the compiled files inside the new folder 
  ```javascript
  entry: './src/app.js',
  output: {
    path: path.join(__dirname, 'public', 'dist'), //absolute path to the public/dist folder
    filename: 'bundle.js'
  },
  ```
- The devServer never writes files in disk instead keeps them in memory, we need a new property to set the new dist folder
  ```javascript
  devServer: {
    contentBase: path.join(__dirname, 'public') ,
    historyApiFallback: true,
    publicPath: '/dist/'
  }
  ```
- Check devServer by running yarn r```un dev-server```. Head over to localhost:8080
- Check prod by running ```yarn run build:prod```, check new dist folder, then ```yarn start``` and head over to localhost:3000
- We can simplify .gitignore file
  ```javascript
  node_modules/
  public/dist/
  ```
- Commit and push
  - If there're no new files, just modified files we can use a shortcut 
  ```javascript
  git commit -am "setup dev devDependencies and dist folder"
  git push //to github
  git push heroku master
  ```
  
# New feature workflow
- yarn run dev-server
- yarn run test --watch
- change date format
  ```javascript
  //ExpenseListItem.js
  import moment from 'moment'
  {moment(createdAt).format('MMM Do, YYYY')}
  ``` 
- change currency format
  - http://numeraljs.com
  - Also valid to use Intl.NumberFormat
  ```javascript
  //ExpenseListItem.js
  yarn add numeral
  import numeral from 'numeral'
  {numeral(amount / 100).format('$0,0.00')} 
  ```
- Commit and push changes
  ```javascript
  git status
  git commit -a -m "setup formatting for amount and createdAt"
  git status
  git push //to github
  git push heroku master 
  ```

## Adding Total selector
  ```javascript
  //expenses-total.js
  //returns the sum of all amounts in the expenses array
  export default (expenses) => expenses
    .map((expense) => expense.amount)
    .reduce((sum, value) => sum + value, 0)

  /* The reduce() method reduces the array to a single value.
    The reduce() method executes a provided function for each value of the array (from left-to-right).
    The return value of the function is stored in an accumulator
    Note: reduce() does not execute the function for array elements without values.
    array.reduce(function(total, currentValue, currentIndex, arr), initialValue)
  */
  //Equivalent function before simplification
  export default (expenses) => {
    return expenses.map((expense) => {
      return expense.amount
    }).reduce((acc, currValue) => {
      return acc + currValue
    }, 0)
  } 
  ```
  ```javascript
  //Alternative way 1
  import _ from "lodash";
  export const selectExpensesTotal = expenses => _.sumBy(expenses, "amount");
  //Alternative way 2
  export default (expenses) => {
  return expenses.reduce((total, expense) => total + expense.amount, 0);
  };
  ```
  ```javascript
  //expenses-total.test.js
  import selectExpensesTotal from '../../selectors/expenses-total'
  import expenses from '../fixtures/expenses'

  test('should return 0 if no expenses', () => {
    const result = selectExpensesTotal([])
    expect(result).toBe(0)
  })

  test('should correctly add up a single expense', () => {
    const result = selectExpensesTotal([expenses[0]])
    expect(result).toBe(expenses[0].amount)
  })

  test('should correctly add up multiple expenses', () => {
    const expectedSum = 114195
    const result = selectExpensesTotal(expenses)
    expect(result).toBe(expectedSum)
  })
  ```

## New component ExpensesSummary
  ```javascript
  //ExpensesSummary.js
  import React from 'react'
  import { connect } from 'react-redux'
  import selectExpenses from '../selectors/expenses'
  import selectExpensesTotal from '../selectors/expenses-total'
  import numeral from 'numeral'

  export const ExpensesSummary = ( {expenseCount, expensesTotal}) => {
    const expenseWord = expenseCount === 1 ? 'expense' : 'expenses'
    const formattedExpensesTotal = numeral(expensesTotal / 100).format('$0,0.00')
    return (
      <div>
        <h1>Viewing {expenseCount} {expenseWord} totalling {formattedExpensesTotal}</h1>
      </div>
    )
  }

  const mapStateToProps = (state) => {
    const visibleExpenses = selectExpenses(state.expenses, state.filters)
    return {
      expenseCount: visibleExpenses.length,
      expensesTotal: selectExpensesTotal(visibleExpenses)
    }
  }

  export default connect(mapStateToProps)(ExpensesSummary)

  //ExpensesSummary.test.js
  import React from 'react'
  import { shallow } from 'enzyme'
  import { ExpensesSummary } from '../../components/ExpensesSummary'

  test('should correctly render ExpensesSummary with one expense', () => {
    const wrapper = shallow(<ExpensesSummary expenseCount={1} expensesTotal={235} />)
    expect(wrapper).toMatchSnapshot()
  })

  test('should correctly render ExpensesSummary with multiple expenses', () => {
    const wrapper = shallow(<ExpensesSummary expenseCount={23} expensesTotal={1141952323} />)
    expect(wrapper).toMatchSnapshot()
  })

  //ExpenseDashboardPage.js
  import React from 'react'
  import ExpenseList from './ExpenseList'
  import ExpenseListFilters from './ExpenseListFilters'
  import ExpensesSummary from './ExpensesSummary'

  const ExpenseDashboardPage = () => (
    <div>
      <ExpensesSummary />
      <ExpenseListFilters />
      <ExpenseList />
    </div>
  );

  export default ExpenseDashboardPage
  ```

# Firebase
- Firebase is a database solution released by google
- https://firebase.google.com
- Firebase has a whole bunch of features built in. We're using two big ones the real time database and their authentication system for web
- At its core the firebase database is a NoSQL database that resembles a javascript object, so we have various key-value pairs where our data is stored, the key is a string and the value can be a string, number, boolean , array or another object

## Connecting to the database
- Create a new Realtime database in test mode (not authenticated) from the web interface. The rules will be in true for read and write but later will be locked in
- Have a look at the information to add Firebase to your App, they provide a script and a config object, we'll use an npm module instead of the script
- We will also break some things into environment variables
  ```javascript
  //original script from Firebase Web
  <script src="https://www.gstatic.com/firebasejs/5.5.9/firebase.js"></script>
  <script>
    // Initialize Firebase
    var config = {
      apiKey: "AIzaSyDL4ArMA00IoAxxHCe7ITLta5zyHN7Wz2c",
      authDomain: "expensify-rocio.firebaseapp.com",
      databaseURL: "https://expensify-rocio.firebaseio.com",
      projectId: "expensify-rocio",
      storageBucket: "expensify-rocio.appspot.com",
      messagingSenderId: "228521581747"
    };
    firebase.initializeApp(config);
  </script>
  ```
- Install Firebase ```yarn add firebase```
- Create file ```src/firebase/firebase.js``` in this file we're going to connect to the database and then other files in our project can use the connection by just importing this file
- This new way of import takes all of the named exports from firebase and dumps them on a new variable called firebase. We can do the same with our code
  ```javascript
  import * as firebase from 'firebase'
  ```
- We could do the same for our 3 named exports in src/actions/expenses
  ```javascript
  import * as expensesActions from '../actions/expenses'
  //and then we'll have access to
  expensesActions.addExpense()
  expensesActions.editExpense()
  expensesActions.removeExpense()
  ```
- This is just a different way to grab all those named exports instead of manually adding the names in the import statement. This is the recommended way to use firebase
- Connection File
  ```javascript
  //connect to the database
  //take all of the named exports from firebase and dumps them on a new variable called firebase
  import * as firebase from 'firebase'

  //from firebase website
  const config = {
    apiKey: "AIzaSyDL4ArMA00IoAxxHCe7ITLta5zyHN7Wz2c",
    authDomain: "expensify-rocio.firebaseapp.com",
    databaseURL: "https://expensify-rocio.firebaseio.com",
    projectId: "expensify-rocio",
    storageBucket: "expensify-rocio.appspot.com",
    messagingSenderId: "228521581747"
  };

  //initialize app
  firebase.initializeApp(config)

  //test the connection
  firebase.database().ref().set({
    name: 'Rocio Pena'
  })
  ```
- Import in in app.js so that the file runs when the server starts
  ```javascript
  import './firebase/firebase'
  ```
- Start the development server ```yarn run dev-server```
- Check the app is up and running
- Check the database in the firebase web

## Writing to the database
- We can add more complex data structures like this
  ```javascript
  //test the connection
  firebase.database().ref().set({
    name: 'Rocio Pena',
    age: 38,
    isSingle: false,
    location: {
      city: 'Melbourne',
      country: 'Australia'
    }
  })
  ```
- Breaking down the test connection code
  - ```firebase.database()``` allow us to interact with the database component/feature of firebase. There're many others such as hosting, test labs, functions etc. We'll later use firebase.auth() to interact with the authentication component
  - ```ref()``` is short for reference and this gives us a reference to an specific part of our database, similar to tables in SQL or collections in MongoDB. If we don't pass anything in we'll get a reference to the root od the database 
  - ```set()``` can be called on a reference to set the value for that reference. It doesn't have to be an object, we can pass a string or any other data type. ```firebase.database().ref().set('This is my data')```
  - It's important to note that ```set``` is going to take what you pass to it and it's going to completely wipe the original reference value and set it equal to that new value. It completely overrides the database.
  - To change a particular part of the object we need to get a reference to the location we want to change
  ```javascript
  firebase.database().ref().set({
    name: 'Rocio Pena',
    age: 38,
    isSingle: false,
    location: {
      city: 'Melbourne',
      country: 'Australia'
    }
  })
  //gets a reference of age
  firebase.database().ref('age').set(39)
  ```
- To get a reference of a nested property we use ```/```
  ```javascript
  firebase.database().ref('location/city').set('Sydney')
  ```
- To add a new object inside the existing object
  ```javascript
  firebase.database().ref('attributes').set({
    height: 156,
    weight: 60
  })
  ```
- Calls to ```set``` are asynchronous so we need a way to know when the data has actually changed, we use ```promises``` for that

## Promises
- Used when we have long tasks such as calls to APIs, changes to DB etc
- A promise can either be resolved or rejected never both. We can do several things with the promise result
  ```javascript
  const promise = new Promise((resolve, reject) => {
    //simulate a long running asynchronous task
    setTimeout(() => {
      resolve('This is my resolved data')
    }, 1500) //waits 1 second and a half
  })
  console.log('before')
  //fires when the promise was resolved
  promise.then((data) => {
    console.log('1', data)
  })
  //we can do several things when the promise is resolved
  promise.then((data) => {
    console.log('2', data)
  })
  console.log('after')
  ```
- We can only resolve or reject a promise a single time
- We can only pass a single argument to resolve and reject, if we need more than a single piece of information we can resolve an object 
  ```javascript
  const promise = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({
        name: 'Rocio',
        age: 38
      })
    }, 1500)
  })
  console.log('before')
  promise.then((data) => {
    console.log('1', data)
  })
  console.log('after')
  ```
- We use ```resolve``` when things went well, when there was an error we use ```reject```
  ```javascript
  const promise = new Promise((resolve, reject) => {
    setTimeout(() => {
      reject('Something went wromg')
    }, 1500)
  })
  console.log('before')
  promise.then((data) => {
    console.log('1', data)
    //catch fires when the promise rejects
  }).catch((error) => {
    console.log('error:', error)
  })
  console.log('after')
  ```
- There are a lot of alternatives to the promise syntax
  - ```then``` can actually take two arguments, two functions and use this instead of catch. If we pass a second argument to then, it would be taken as the error handler. This approach might be confusing
  ```javascript
  promise.then((data) => {
    console.log('1', data)
    //catch fires when the promise rejects
  }, (error) => {
    console.log('error:', error)
  })
  ```
- When working with promises we usually work with the ```then/catch``` side of things. Very rarely we'll find ourselves creating our own promises. Most of the time the promises are going to be created by the library you are using, for example firebase. What we're going to do is to attach handlers; things to run when the promise either resolves or rejects

## Promises with Firebase

### Add data
- ```set``` returns a promise so we can continue chaining methods ```then``` or ```catch```. To test the error change the rules to write a write to false in firebase.
  ```javascript
  firebase.database().ref().set({
    name: 'Rocio Pena',
    age: 38,
    isSingle: false,
    location: {
      city: 'Melbourne',
      country: 'Australia'
    }
  }).then(() => { //then returns nothing
    console.log('Data is saved')
  }).catch((error) => {
    console.log('This failed. ', error)
  })
  console.log('I made a request to change the data')
  ```
- Firebase documentation https://firebase.google.com/docs/
- Ref and Set https://firebase.google.com/docs/reference/js/firebase.database.Reference#set

### Remove data
  ```javascript
  //removes the isSingle field
  firebase.database().ref('isSingle').remove()
    .then(() => {
      console.log('Data removed')
    })
    .catch((error) => {
      console.log('Error removing data', error)
    })
  ```
- There's an alternative way to remove data that is using set with a null value. This is less explicit than remove so we'll stick with remove
  ```javascript
  //removing with set
  firebase.database().ref('isSingle').set(null)
  ```

### Update data
- We can use the update method to change data more efficiently especially when we need to change several pieces of data at the same time. Update has to be called with an object because the whole idea is to update multiple things at once
  ```javascript
  firebase.database().ref().update({
    name: 'Juana de Arco',
    age: 101
  })
  ```
- Update is not only for existing elements, we can also add new properties to the object
  ```javascript
  firebase.database().ref().update({
    name: 'Juana de Arco',
    age: 101,
    job: 'Software Developer'
  })
  ```
- We can also delete properties with update by setting null as its value
  ```javascript
  firebase.database().ref().update({
    name: 'Juana de Arco',
    age: 101,
    job: 'Software Developer',
    isSingle: null
  })
  ```
- When updating nested objects we need to provide a reference to the nested object as the key and the new value as value. For this we need to use forward slash (/) and because this make the javascript object invalid we wrap it up with quotes
  ```javascript
  firebase.database().ref().set({
    name: 'Rocio Pena',
    age: 38,
    job: 'Software developer',
    location: {
      city: 'Melbourne',
      country: 'Australia'
    }
  }).then(() => { //then returns nothing
    console.log('Data is saved')
  }).catch((error) => {
    console.log('This failed. ', error)
  })

  firebase.database().ref().update({
    job: 'Manager',
    //reference location as they key and the new value
    'location/city': 'Sydney' //child location
  })
  ```
- Update also supports promises
  ```javascript
  //Challenge
  firebase.database().ref().set({
    name: 'Rocio Pena',
    age: 38,
    stressLevel: 6,
    job: {
      title: 'Software developer',
      company: 'Google'
    },
    location: {
      city: 'Melbourne',
      country: 'Australia'
    }
  }).then(() => { //then returns nothing
    console.log('Data is saved')
  }).catch((error) => {
    console.log('This failed. ', error)
  })

  firebase.database().ref().update({
    stressLevel: 9,
    'job/company': 'Amazon',
    'location/city': 'Sydney'
  }).then(() => { //then returns nothing
    console.log('Data updated')
  }).catch((error) => {
    console.log('Update failed. ', error)
  })
  ```

### Fetch data
There're two big ways we can fetch data

#### Fetch data a single time getting the object back
- ```ref()``` gets all data, we can define what we want eg. ```database.ref('name')``` or ```database.ref('job/company')```
- ```once``` receives as the first and only argument the eventType
- ```once``` returns a promise and we use that promise to do something when the data comes back or when it's an error fetching the data
- Unlike set, remove and update, with once we do get an argument back. This is called a snapshot and with the method val() we have access to the data
  ```javascript
  //Fetch data
  const database = firebase.database()
  database.ref()
    .once('value')
    .then((snapshot) => {
      const val = snapshot.val()
      console.log(val)
    })
    .catch((error) => {
      console.log('Error fetching data', error)
    })
  ```
  
#### Fetch data and subscribe to changes
- Fetch data and set up a subscription so that the app will be notified if the data even changes
- For this case we use the method ```on```
- As the second argument, after eventType we need to pass a callback function. There's not possible to use a promise here because promises resolve or reject only once and because we're subscribing to changes we need to run this callback function multiple times, once every time the data changes
  ```javascript
  const database = firebase.database()
  database.ref().on('value', (snapshot) => {
    console.log(snapshot.val())
  })
  ```
- We can test changes by editing data directly in Firebase
- We can also simulate a change with setTimeout
  ```javascript
  const database = firebase.database()
  //get data and subscribe
  database.ref().on('value', (snapshot) => {
    console.log(snapshot.val())
  })

  setTimeout(() => {
    database.ref('age').set(24)
  }, 3500) //3.5 seconds
  ```
- We can also unsubscribe with the ```off``` method
  ```javascript
  database.ref().on('value', (snapshot) => {
    console.log(snapshot.val())
  })

  setTimeout(() => {
    database.ref('age').set(38)
  }, 3500) //3.5 seconds

  //remove all subscriptions
  setTimeout(() => {
    database.ref().off()
  }, 7000)

  //data still changes but our app is not notified
  setTimeout(() => {
    database.ref('age').set(40)
  }, 10500)
  ```
- ```off``` with no parameters will remove all subscriptions. If we have multiple subscriptions and want to unsubscribe one, we need to pass the same function we used when we subscribed
- One way to do this is by pulling out the function to its own const
  ```javascript
  const onValueChange = (snapshot) => {
    console.log(snapshot.val())
  }
  //use the function defined before as a variable
  database.ref().on('value', onValueChange)

  setTimeout(() => {
    database.ref('age').set(38)
  }, 3500) //3.5 seconds

  setTimeout(() => {
    //use the same function defined before as a variable
    database.ref().off(onValueChange)
  }, 7000)
  ```
- Another way to do this and the one we'll follow is to to pass to the ```off``` function the return value of the ```on``` function which is the callback function itself.
  ```javascript
  //on returns the callback function
  const onValueChange = database.ref().on('value', (snapshot) => {
    console.log(snapshot.val())
  })

  setTimeout(() => {
    database.ref('age').set(38)
  }, 3500) //3.5 seconds

  setTimeout(() => {
    //use the return value of on
    database.ref().off(onValueChange)
  }, 7000)
  ```
- We can also handle errors fetching data by adding a third argument to the method on
  ```javascript
  const onValueChange = database.ref().on('value', (snapshot) => {
    console.log(snapshot.val())
  }, (error) => {
    console.log('Error with data fetching', error)
  })
  ```

### Arrays in Firebase
- Firebase does not support arrays. At least not in the same way we know them in Javascript
- Instead the data structure will look like this
  ```javascript
  const firebaseNotes = {
    notes: {
      some_id: {
        title: 'First note',
        body: 'This is my note'
      },
      some_other_id: {
        title: 'Another note',
        body: 'Note Body'
      }
    }
  }
  ```
- The method push will allow us to add objects and it will generate those unique ids
  ```javascript
  const database = firebase.database()
  database.ref('notes').push({
    title: 'To Do',
    body: 'Go for a run'
  })
  database.ref('notes').push({
    title: 'Course Topics',
    body: 'React Native, Angular, Python'
  })
  ```
- Then we can reference any of the objects by its unique id and execute any of the methods we saw before update, remove, set etc
  ```javascript
  //update
  database.ref('notes/-LS8uTcHQm_vE2WEDzLq').update({
    body: 'Buy food'
  })
  //remove
  database.ref('notes/-LS8uTcHQm_vE2WEDzLq').remove()
  ```
- When we fetch lists from Firebase we'll need to do a bit of conversion to get the data the way our app needs it. By default using snapshot.val() we get:
  ```javascript
  database.ref('expenses')
    .once('value')
    .then((snapshot) => {
      console.log(snapshot.val())
    })
  ```
![Firebase Data](./img/firebase-data.png)
- We can use the ```forEach``` method on snapshot to manipulate the data
  ```javascript
  database.ref('expenses')
    .once('value')
    .then((snapshot) => {
      const expenses = []
      snapshot.forEach((childSnapshot) => {
        expenses.push({
          //key is the unique id firebase generated
          //we'll use it as the expense id
          id: childSnapshot.key,
          ...childSnapshot.val()
        })
      })
      console.log(expenses)
    })
  ```
 ![Expenses Array](./img/expenses-array.png) 
- Challenge subscribe to changes
  ```javascript
  //Challenge
  database.ref('expenses')
    .on('value', (snapshot) => {
      const expenses = []
      snapshot.forEach((childSnapshot) => {
        expenses.push({
          id: childSnapshot.key,
          ...childSnapshot.val()
        })
      })
      console.log(expenses)
    }, (error) => {
      console.log('Error fetching data', error)
    })
  ```
- There are other eventTypes aside from 'value'
- Using ```child_removed``` for instance is convenient so we don't have to manually figure out what changed
  ```javascript
  database.ref('expenses').on('child_removed', (snapshot) => {
    console.log(snapshot.key, snapshot.val())
  })
  ```
- ```child_changed``` is another common one
  ```javascript
  database.ref('expenses').on('child_changed', (snapshot) => {
    console.log(snapshot.key, snapshot.val())
  })
  ```
- ```child_added ```works a bit different than expected, it triggers once for each element already in the database and again for each new one
  ```javascript
  database.ref('expenses').on('child_added', (snapshot) => {
    console.log(snapshot.key, snapshot.val())
  })
  ```
  
# Firebase with Redux

## Asynchronous Redux Actions
- Instead of only dispatching redux actions we're going to save the data to firebase first and then dispatch the action to save data in our redux store
- Component shouldn't be concerned about our database choice etc, they are just representational. A better place to include our firebase code is in actions
- With the current logic, these are the tasks our app follows to update the redux store
  - component calls action generator
  - action generator returns an object
  - component dispatches object
  - redux store changes
- To add changes to firebase we need to change a few things
  - component calls action generator
  - action generator returns a **function**
  - component dispatches **function** (redux by default does not allow you to dispatch functions. So we need to set up a module which is a piece of redux middleware that's going to add support for it)
  - function runs (we can put whatever we want in the function, for instance our push method to firebase, we can also put in here regular actions that return an object and that will manipulate the redux store)
- ```redux-thunk``` is the module we'll use to add support for dispatching functions. We need to install the module and also make some changes to our store. https://github.com/reduxjs/redux-thunk
  ```javascript
  yarn add redux-thunk
  ```
  ```javascript
  //configureStore.js
  //applyMiddleware let you add a middleware to your store such as thunk
  //compose Composes functions from right to left. Each function is expected to accept a single parameter. 
  //Its return value will be provided as an argument to the function standing to the left, and so on. 
  //The exception is the right-most argument which can accept multiple parameters, as it will provide 
  //the signature for the resulting composed function. https://redux.js.org/api/compose
  import { createStore, combineReducers, applyMiddleware, compose } from 'redux'
  import expensesReducer from '../reducers/expenses'
  import filtersReducer from "../reducers/filters";
  //import thunk
  import thunk from 'redux-thunk'

  //this is because we're using react dev-tools
  const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose
  // Store creation
  export default () => {
    const store = createStore(
      combineReducers({
        expenses: expensesReducer,
        filters: filtersReducer
      }),
      //if we weren't using react dev-tools this will be simply applyMiddleware(thunk)
      composeEnhancers(applyMiddleware(thunk))
    )
    return store
  }
  ```
- Add exports to firebase.js
  ```javascript
  import * as firebase from 'firebase'

  //from firebase website
  const config = {
    apiKey: "AIzaSyDL4ArMA00IoAxxHCe7ITLta5zyHN7Wz2c",
    authDomain: "expensify-rocio.firebaseapp.com",
    databaseURL: "https://expensify-rocio.firebaseio.com",
    projectId: "expensify-rocio",
    storageBucket: "expensify-rocio.appspot.com",
    messagingSenderId: "228521581747"
  };

  //initialize app
  firebase.initializeApp(config)

  const database = firebase.database()

  export { firebase, database as default }
  ```
- Change all the add expense functionality in actions/expenses.js
  ```javascript
  // ADD_EXPENSE
  export const addExpense = (expense) => ({
    type: 'ADD_EXPENSE',
    expense
  })

  export const startAddExpense = (expenseData = {}) => {
    //returning a function for asynchronous redux actions
    //this is possible only because we set up thunk, by default this wouldn't work
    //this function gets called internally by redux and gets called with dispatch
    return (dispatch) => {
      //alternative destructuring and setting defaults
      const {
        description = '',
        note = '', 
        amount = 0, 
        createdAt = 0
      } = expenseData
      const expense = { description, note, amount, createdAt }
      //save expense to the database
      //then gets called with a reference to the pushed element
      database.ref('expenses').push(expense).then((ref) => {
        //make changes in the store
        dispatch(addExpense({
          //changing id from moment to the one firebase generates 
          id: ref.key,
          ...expense
        }))
      })
    }
  }
  ```
- Change AddExpensePage to dispatch the new startAddExpense instead of addExpense
  ```javascript
  import React from 'react'
  import ExpenseForm from './ExpenseForm'
  import { connect } from 'react-redux'
  import { startAddExpense } from '../actions/expenses'

  export class AddExpensePage extends React.Component {
    onSubmit = (expense) => {
      //change from addExpense to startAddExpense
      this.props.startAddExpense(expense)
      this.props.history.push('/')
    }
    render() {
      return (
        <div>
          <h1>Add Expense</h1>
          <ExpenseForm onSubmit={this.onSubmit}/>
        </div>
      )
    }
  }
  const mapDispatchToProps = (dispatch) => ({ 
    //change to startAddExpense
    startAddExpense: (expense) => dispatch(startAddExpense(expense)) 
  })
  export default connect(undefined, mapDispatchToProps)(AddExpensePage)
  ```
- Start dev-server and test ```yarn run dev-server``` (clean firebase first)

## Testing Async Redux actions
- Start test suite ```yarn test --watch```
- Fix AddExpensePage.test.js by changing addExpense to startAddExpense
  ```javascript
  import React from 'react'
  import { shallow } from 'enzyme'
  import { AddExpensePage } from '../../components/AddExpensePage'
  import expenses from '../fixtures/expenses'

  let startAddExpense, history, wrapper

  beforeEach(() => {
    startAddExpense = jest.fn()
    history = { push: jest.fn() }
    wrapper = shallow(<AddExpensePage startAddExpense={startAddExpense} history={history} />)
  })

  test('should render AddExpensePage correctly', () => { 
    expect(wrapper).toMatchSnapshot()
  })

  test('should handle onSubmit', () => {
    wrapper.find('ExpenseForm').prop('onSubmit')(expenses[1])
    expect(history.push).toHaveBeenCalledWith('/')
    expect(startAddExpense).toHaveBeenCalledWith(expenses[1])
  })
  ``` 
- The test 'should setup add expense action object with default values' fails and this is now the responsibility of 'startAddExpense'
- Now, our app do not generate an unique id, instead we use the firebase one. Fix test
  ```javascript
  test('should setup add expense action object with provided values', () => { 
    const action = addExpense(expenses[2])
    expect(action).toEqual({
      type: 'ADD_EXPENSE',
      expense: expenses[2]
    })
  })
  ```
- We now add two nes test cases for the async actions. First one is 'should add expense to database and store'
- For this test we care that the database was updated and that the correct action was dispatched

### Check dispatched action
- Figuring out if an action was dispatched is not easy with the regular redux store, lucky for us there's a test module for the redux store that makes it super easy to mock called ```redux-mock-store```. https://github.com/dmitry-zaets/redux-mock-store
- We're going to go through the process of creating a mock store, dispatching things to it using ```store.dispatch``` and then using the method store.getActions to get the actions back and assert that the correct ones were dispatched
- Install library ```yarn add redux-mock-store```
- Restart test suite ```yarn test --watch```
- Promise Chaining
  ```javascript
  //playground
  //promise chaining
  const promise = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({
        name: 'Rocio',
        age: 38
      })
      // reject('Something went wrong')
    }, 1500)
  })

  console.log('before')
  promise.then((data) => {
    console.log('1', data)
  }).then(() => { //we can chain multiple then - things to do when the promise resolves
    console.log('Does this run?')
  }).catch((error) => {
    console.log('error:', error)
  })
  console.log('after')
  ```
- The second ```then``` doesn't get any data unless we return some data from the first one
  ```javascript
  //playground
  console.log('before')
  promise.then((data) => {
    console.log('1', data)
    return 'some data'
  }).then((str) => { //we can chain multiple then - things to do when the promise resolves
    console.log('Does this run?', str)
  }).catch((error) => {
    console.log('error:', error)
  })
  console.log('after')
  ```
- Adding test
  ```javascript
  //add test cases for the async Redux actions  
  //we need to tell jest this test is async bt passing done
  test('should add expense to database and store', (done) => {
    const store = createMockStore({}) //passing the default data
    const expenseData = {
      description: 'Mouse',
      amount: 3000,
      note: 'This one is better',
      createdAt: 1000
    }
    store.dispatch(startAddExpense(expenseData)).then(() => {
      expect(1).toBe(1)
      //all assertions will wait until this specific point in time
      done()
    })
  })
  ```
- We also needed to change our async code to return a promise
  ```javascript
  export const startAddExpense = (expenseData = {}) => {
    //returning a function for asynchronous redux actions
    //this is possible only because we set up thunk, by default this wouldn't work
    //this function gets called internally by redux and gets called with dispatch
    return (dispatch) => {
      //alternative destructuring 
      const {
        description = '',
        note = '', 
        amount = 0, 
        createdAt = 0
      } = expenseData
      const expense = { description, note, amount, createdAt }
      //save expense to the database
      //we set up this to return a promise to be able to test it
      return database.ref('expenses').push(expense).then((ref) => {
        //make changes in the store
        dispatch(addExpense({
          id: ref.key,
          ...expense
        }))
      })
    }
  }
  ```
- We finally do the assertions
  ```javascript
  //add test cases for the async Redux actions  
  //we need to tell jest this test is async bt passing done
  test('should add expense to database and store', (done) => {
    const store = createMockStore({}) //passing the default data
    const expenseData = {
      description: 'Mouse',
      amount: 3000,
      note: 'This one is better',
      createdAt: 1000
    }
    store.dispatch(startAddExpense(expenseData)).then(() => {
      const actions = store.getActions() //returns array of actions 1 in our case
      expect(actions[0]).toEqual({
        type: 'ADD_EXPENSE',
        expense: {
          id: expect.any(String),
          ...expenseData
        }
      })
      //all assertions will wait until this specific point in time
      done()
    })
  })
  ```

### Checking the database was updated
- Building upon the previous code
  ```javascript
  //add test cases for the async Redux actions  
  //we need to tell jest this test is async bt passing done
  test('should add expense to database and store', (done) => {
    const store = createMockStore({}) //passing the default data
    const expenseData = {
      description: 'Mouse',
      amount: 3000,
      note: 'This one is better',
      createdAt: 1000
    }
    store.dispatch(startAddExpense(expenseData)).then(() => {
      const actions = store.getActions() //returns array of actions 1 in our case
      expect(actions[0]).toEqual({
        type: 'ADD_EXPENSE',
        expense: {
          id: expect.any(String),
          ...expenseData
        }
      })
      //we have access to the id of the created expense
      database.ref(`expenses/${actions[0].expense.id}`).once('value').then((snapshot) => {
        expect(snapshot.val()).toEqual(expenseData)
        //all assertions will wait until this specific point in time
        done()
      })
    })
  })
  ```
- There's a better way to do this and avoid using many callbacks by using promise chaining
- In promise chaining we can also return another promise. If we return a promise the next ```then``` callback is that promise's success case
  ```javascript
  //playground
  console.log('before')
  promise.then((data) => {
    console.log('1', data)
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve('This is my other promise')
      }, 1500)
    })
  }).then((str) => { //this then will be the success from the previous return
    console.log('Does this run?', str)
  }).catch((error) => {
    console.log('error:', error)
  })
  console.log('after')
  ```
- Return a promise in our code
  ```javascript
  test('should add expense to database and store', (done) => {
    const store = createMockStore({}) //passing the default data
    const expenseData = {
      description: 'Mouse',
      amount: 3000,
      note: 'This one is better',
      createdAt: 1000
    }
    store.dispatch(startAddExpense(expenseData)).then(() => {
      const actions = store.getActions() //returns array of actions 1 in our case
      expect(actions[0]).toEqual({
        type: 'ADD_EXPENSE',
        expense: {
          id: expect.any(String),
          ...expenseData
        }
      })
      //this is the promise we're returning
      return database.ref(`expenses/${actions[0].expense.id}`).once('value')
    }).then((snapshot) => { //chaining here
      expect(snapshot.val()).toEqual(expenseData)
      done()
    })
  })
  ```
- Defaults test
  ```javascript
  test('should add expense with defaults to database and store', (done) => {
    const store = createMockStore({}) //passing the default data
    const expenseDefaults = {
      description: '',
      amount: 0,
      note: '',
      createdAt: 0
    }
    store.dispatch(startAddExpense()).then(() => { //no data passed
      const actions = store.getActions() //returns array of actions 1 in our case
      expect(actions[0]).toEqual({
        type: 'ADD_EXPENSE',
        expense: {
          id: expect.any(String),
          ...expenseDefaults
        }
      })
      //this is the promise we're returning
      return database.ref(`expenses/${actions[0].expense.id}`).once('value')
    }).then((snapshot) => { //chaining here
      expect(snapshot.val()).toEqual(expenseDefaults)
      done()
    }) 
  })
  ```
- These tests are actually writing to the database. In next sections we'll create a test database

## Creating a separate database
- There is an environment variable that stores the environment you're currently in. ```process.env.NODE_ENV```. This gets automatically set for us on Heroku. Heroku sets this value equal to the string **production**. It can also be test for testing and undefined for development
- What we're going to do is figure out how we can set this Heroku variable in our test environment. We're going to be changing our script ```test``` in package.json and to make the change work across different OS we need an npm module called cross-env https://www.npmjs.com/package/cross-env
- Install cross-env as a dev dependency
  ```javascript
  yarn add --dev cross-env
  ```
- Change script in package.json. We only change test because for prod this is set by Heroku and the absence of the environment variable tell us it is development
  ```javascript
  "test": "cross-env NODE_ENV=test jest --config=jest.config.json",
  ```
- Create new database in Firebase for testing purposes
- Create two new files .env.test and .env.development. The idea is to break the firebase configuration into pairs KEY=VALUE (don't add quotes). These .env files are not to be committed to the repository
  ```javascript
  //.env.development - Original Database
  FIREBASE_API_KEY=AIzaSyDL4ArMA00IoAxxHCe7ITLta5zyHN7Wz2c
  FIREBASE_AUTH_DOMAIN=expensify-rocio.firebaseapp.com
  FIREBASE_DATABASE_URL=https://expensify-rocio.firebaseio.com
  FIREBASE_PROJECT_ID=expensify-rocio
  FIREBASE_STORAGE_BUCKET=expensify-rocio.appspot.com
  FIREBASE_MESSAGING_SENDING_ID=228521581747

  //.env.test - new Test database created in Firebase
  FIREBASE_API_KEY=AIzaSyB9jpU03FFyHOu71097USXGFE9pIWU1oCo
  FIREBASE_AUTH_DOMAIN=expensify-test-rocio.firebaseapp.com
  FIREBASE_DATABASE_URL=https://expensify-test-rocio.firebaseio.com
  FIREBASE_PROJECT_ID=expensify-test-rocio
  FIREBASE_STORAGE_BUCKET=expensify-test-rocio.appspot.com
  FIREBASE_MESSAGING_SENDING_ID=648575174841  
  ``` 
- Now, we'll take advantage of the environment variable NODE_ENV in webpack.config. We're reading that environment variable and the result here is test, production or development if it's undefined. Here we want to read all the firebase config info depending of the environment. We won't read the variables one by one, instead we'll use the npm module ```dotenv``` which reads those .env files and creates process.env.KEY for each of the values in the file. install it with ```yarn add --dev dotenv```. https://www.npmjs.com/package/dotenv
  ```javascript
  process.env.NODE_ENV = process.env.NODE_ENV || 'development'

  if(process.env.NODE_ENV === 'test') {
    require('dotenv').config({ path: '.env.test' })
  } else if (process.env.NODE_ENV === 'development') {
    require('dotenv').config({ path: '.env.development' })
  }
  ```
- Now all those values are read and put in process.env depending on the environment. However our NODE_ENV variable is not passed down to the client side javascript. If they did it would create a ton of security concerns so what we need to do instead is manually pass those six values down into client side javascript in bundle.js. We'll use a built-in webpack plugin for that
  ```javascript
  const webpack = require('webpack')
  ...
  plugins: [
    CSSExtract,
    new webpack.DefinePlugin({
      //we need to get the string and not the value, something like "'test'" 
      //but adding those double quotes is tedious so we use JSON.stringify 
      'process.env.FIREBASE_API_KEY': JSON.stringify(process.env.FIREBASE_API_KEY),
      'process.env.FIREBASE_AUTH_DOMAIN': JSON.stringify(process.env.FIREBASE_AUTH_DOMAIN),
      'process.env.FIREBASE_DATABASE_URL': JSON.stringify(process.env.FIREBASE_DATABASE_URL),
      'process.env.FIREBASE_PROJECT_ID': JSON.stringify(process.env.FIREBASE_PROJECT_ID),
      'process.env.FIREBASE_STORAGE_BUCKET': JSON.stringify(process.env.FIREBASE_STORAGE_BUCKET),
      'process.env.FIREBASE_MESSAGING_SENDING_ID': JSON.stringify(process.env.FIREBASE_MESSAGING_SENDING_ID)
    })
  ],
  ```
- We also need to change the firebase.js file. Nad that's all for development environment
  ```javascript
  const config = {
    apiKey: process.env.FIREBASE_API_KEY,
    authDomain: process.env.FIREBASE_AUTH_DOMAIN,
    databaseURL: process.env.FIREBASE_DATABASE_URL,
    projectId: process.env.FIREBASE_PROJECT_ID,
    storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
    messagingSenderId: process.env.FIREBASE_MESSAGING_SENDING_ID
  }
  ```
- For test environment we need to add this to src/tests/setupTests.js
  ```javascript
  import DotEnv from 'dotenv'
  DotEnv.config({ path: '.env.test' })
  ```
- We have now development and test all set. In production if we want to set up environment variables we have to use the Heroku command line interface which allow us to set up environment variables on process.ENV, NODE_ENV is automatically set up for us to the string 'production' but we need to set up the rest of the variables
- Commands
  ```javascript
  heroku config //prints out all of your environment variables
  heroku config:set KEY=value //set environment variable and restarts the app
  //we can set multiple separating them by space
  heroku config:unset KEY //remove env var and restarts the app
  ```
- Add these two to gitignore
  ```javascript
  .env.test
  .env.development
  ```
- Commit and push
  ```javascript
  git status
  git add .
  git commit -m "set up test database environment"
  git push
  git push heroku master
  ```

## Fetching expenses
- We'll start setting up some data in actions/expenses.test.js
  ```javascript
  import { startAddExpense, addExpense, removeExpense, editExpense, setExpenses } from '../../actions/expenses'
  ...
  //this will take the expenses we have in fixture and add them to firebase in its format
  beforeEach((done) => {
    const expensesData = {}
    expenses.forEach(({ id, description, note, amount, createdAt }) => {
      expensesData[id] = { description, note, amount, createdAt }
    })
    database.ref('expenses').set(expensesData).then(() => done()) //wait
  })
  ...
  test('should setup set expense action object with data', () => {
    const action = setExpenses(expenses)
    expect(action).toEqual({
      type: 'SET_EXPENSES',
      expenses
    })
  })
  ```
- Run test suite ```yarn run test --watch```
- Add action generator
  ```javascript
  // SET_EXPENSES
  export const setExpenses = (expenses) => ({
    type: 'SET_EXPENSES',
    expenses
  })
  ```
- Change Reducer
  ```javascript
  case 'SET_EXPENSES':
    return action.expenses
  ```
- Change Reducer test
  ```javascript
  test('should set expenses', () => {
    const action = {
      type: 'SET_EXPENSES',
      expenses: [expenses[1]]
    }
    const state = expensesReducer(expenses, action)
    expect(state.length).toBe(1)
    expect(state).toEqual([expenses[1]])
  })
  ```










 


    