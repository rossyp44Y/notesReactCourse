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
