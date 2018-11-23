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

# Adding Total selector
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

# New component ExpensesSummary
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

    