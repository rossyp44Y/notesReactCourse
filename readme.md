> markdown: https://guides.github.com/features/mastering-markdown/  
> Preview: Chrome extension and QuickLook / space key on the file 

React Course - Udemy - Andrew Mead
============================================

# Setting up environment
- Node, npm, yarn
  ```
  node -v / node --version
  npm -v  / npm --version
  yarn --version
  npm install -g yarn

  npm list -g --depth=0  //check modules installed globally
  npm config set prefix ~/npm  //check path for global modules  
  ```
- App  
  http://indecision.mead.io  
  https://github.com/andrewjmead/react-course-2-indecision-app

- VSC extensions
  - Sublime Text Keymap
  - Babel ES6/ES7
  - emmet (autocomplete - built-in VSC)
  - Path Intellisense

- Web Server  
  ```
  yarn global add live-server
  live-server -v
  live-server public (inside indecision-app folder)
  ```

# Hello React
- Using CDN
  ```
  <script src="https://unpkg.com/react@16.0.0/umd/react.development.js"></script>  
  <script src="https://unpkg.com/react-dom@16.0.0/umd/react-dom.development.js"></script>  
  (react-dom is specific for the browser, there's another for VR, native etc)
  ``` 
- JSX vs plain Javascript  
  Browsers do not understand JSX, tools like babel translates it
  - JSX   
    ```const template = <p id="my_p">This is JSX from app.js!</p>``` 
  - JavaScript (after babel using the web 'try it out') 
    ```javascript
    var template = React.createElement(
      "p",
      { id: "my_p" },
      "This is JSX from app.js!"
    );
    ```
  - install babel with presets: react and env (includes es2015, es2016 and es2017)   
    ```terminal
    yarn global add babel-cli@6.24.1
    yarn init
    yarn add babel-preset-react@6.24.1 babel-preset-env@1.5.2
    live-server public
    babel src/app.js --out-file=public/scripts/app.js --presets=env,react --watch 
    (manual process to compile src file)
    ```

# JSX Basics
- with JSX we can only have a single root element. (Usually a wrapper ```<div>```)
  ```terminal
  SyntaxError: src/app.js: Adjacent JSX elements must be wrapped in an enclosing tag (4:50)
  ```
- Some indentation and formatting technique ```(...);``` helps with clarity on the code. When this JSX in compiled, it uses nested React.createElement() calls to replicate the structure.
  ```javascript
  // JSX - JavaScript XML
  const template = (
    <div>
      <h1>Indecision App</h1>
      <p>This is some info</p>
      <ol>
        <li>Item one</li>
        <li>Item two</li>
      </ol>
    </div>
  );
  const appRoot = document.getElementById('app'); //better use getElementById for Ids 
                                                  //and querySelector with others selectors
  ReactDOM.render(template, appRoot);
  ```

# JSX Expressions
- To inject dynamic data into JSX templates we use expressions: ```{ }```. 
- We can render strings and numbers but not complete objects. We can use object properties though. (*a bit ahead but we can also render lists*)
  ```javascript
  const app = {
    title: 'Indecision App',
    subtitle: 'Are you feeling lucky?',
    items: ['Item one', 'Item two']
  };

  const template = (
    <div>
      <h1>{app.title}</h1>
      <p>{app.subtitle}</p>
      <ol>
        {app.items.map((element) => <li>{element}</li>)}
      </ol>
    </div>
  );
  const appRoot = document.getElementById('app');
  ReactDOM.render(template, appRoot);
  ```

# JSX Conditional Rendering
- To achieve conditional rendering we can combine truthy/falsy values with:
  - if statements
  - ternary operator (this is an expression not statement)
  - logical and operator ```(&&)```

## If statement  
- Calling a function is an expression but defining an if statement is not.
- Example: show location if it exists, otherwise show 'unknown'
  ```javascript
  const userName = 'Rocio Pena';
  const user = {
    age: 38,
    location: ''
  };
  const getLocation = () => user.location || 'Unknown' 
  const templateTwo = (
    <div>
      <h1>{userName.toUpperCase() + '!'}</h1>
      <p>Age: {user.age}</p>
      <p>Location: {getLocation()}</p>
    </div>
  );
  const appRoot = document.getElementById('app');
  ReactDOM.render(templateTwo, appRoot);
  ```
- JSX expressions can be used inside another JSX expression. e.g. ```{<h3>my h3</h3>}```. This is useful because we can set a function that returns a JSX expression, and call that function inside ```{}```.
- ```undefined``` is the implicit return from functions.
- If an expression resolves to ```undefined```, nothing will be shown. Not even empty or hidden tags. Same for ```true```, ```false``` and ```null```.
- Example: if there's no location, do not show the location ```<p>Location: Melbourne</p>```. Hide the paragraph tah all together.
  ```javascript
  const user = {
    name: 'Rocio',
    age: 38,
    location: ''
  };
  const getLocation = (location) => {
    if(location) {
      return <p>Location: {location}</p>
    } //this function returns location if exists or undefined
  }
  const templateTwo = (
    <div>
      <h1>{user.name}</h1>
      <p>Age: {user.age}</p>
      {getLocation(user.location)} //this will be undefined if location does not exist or it is empty
    </div>
  );
  const appRoot = document.getElementById('app');
  ReactDOM.render(templateTwo, appRoot);
  ```

## Ternary Operator
- ```true  ? 'Rocio' : 'Anonymous'``` // returns 'Rocio'
- ```false ? 'Rocio' : 'Anonymous'``` // returns 'Anonymous'
  ```javascript
  const user = {
    age: 38,
    location: ''
  };
  const templateTwo = (
    <div>
      <h1>{user.name ? user.name : 'Anonymous'}</h1>
      <p>Age: {user.age}</p>
    </div>
  );
  const appRoot = document.getElementById('app');
  ReactDOM.render(templateTwo, appRoot);
  ```

## Logical and operator
- ```true  && 'SomeAge' ``` // returns 'SomeAge'
- ```false && 'SomeAge' ``` // returns false
- If first operator is truthy, uses the second. If first operator is falsy, uses false.
- Example: show Age information only if user is 18 or older
  ```javascript
  const user = {
    age: 17,
    location: ''
  };
  const templateTwo = (
    <div>
      <h1></h1>
      {user.age >= 18 && <p>Age: {user.age}</p>}
      // if user.age >=18 uses second part
      // if user.age <18 expression evaluates to false and it's ignored
    </div>
  );
  const appRoot = document.getElementById('app');
  ReactDOM.render(templateTwo, appRoot);
  ```
- Condition can be more robust: ```{(user.name && user.age >= 18) && <p>Age: {user.age}</p>}```

## All together
  ```javascript
  const user = {
    name: 'Rocio',
    age: 20,
    location: 'Melbourne',
  };
  const getLocation = (location) => {
    if(location) {
      return <p>Location: {location}</p>
    } // return undefined if there's no location
  }
  const templateTwo = (
    <div>
      <h1>{user.name ? user.name : 'Anonymous'}</h1>
      {(user.age && user.age >= 18) && <p>Age: {user.age}</p>}
      {getLocation(user.location)}
    </div>
  );
  const appRoot = document.getElementById('app');
  ReactDOM.render(templateTwo, appRoot);
  ```

# Var vs. let/const
- ```var``` based variables are **function scoped**. That means that vars created inside an if statement are accessible outside the if statement.
- ```let``` and ```const``` based variables are **block level scoped**. They are accessible only inside the 'if' curly braces.
- As a good practice start all variables off as ```const``` and change it to ```let``` if needed. _var_ never :smirk:

# Functions vs. Arrow functions - Syntax
```javascript
// Named function
function square (x) {
  return x * x
}
console.log('Named function', square(2))

// Anonymous function
const square2 = function (x) {
  return x * x
}
console.log('Anonymous function', square2(5))

// Arrow function
const squareArrow = (x) => {
  return x * x
}
console.log('Arrow function', squareArrow(8))

// Arrow function with shorthand syntax
const squareArrow2 = (x) => x * x
console.log('Arrow function with shorthand syntax', squareArrow2(9))
```

# Functions vs. Arrow functions - Deep dive
## arguments object
```javascript
// arguments object - no longer bound with arrow functions
const add = function (a, b) {
  console.log(arguments) //38, 3, 100, ...
  return a + b
}
console.log(add(38, 3, 100)) //41

const add2 = (a, b) => {
  console.log(arguments) //error - can't find variable arguments
  return a + b
}
console.log(add2(38, 3, 100)) //not executed due to error
```
## this object
```javascript
// 'this' keyword - no longer bound
const user = {
  name: 'Rocio',
  cities: ['Colombia', 'Venezuela', 'USA', 'Australia'],
  printPlacesLived: function () {
    console.log(this.name) //Rocio
    console.log(this.cities) //["Colombia", "Venezuela", "USA", "Australia"]
    this.cities.forEach(function (city) {
    console.log(`${this.name} has lived in ${city}`)  // error, undefined is not an object
    })                                                // there's no bound 'this' value
  }
}
user.printPlacesLived();

// workaround
const user2 = {
  name: 'Rocio',
  cities: ['Colombia', 'Venezuela', 'USA', 'Australia'],
  printPlacesLived: function () {
    const that = this;
    //with an anonymous function, 'this' is not bound...as a workaround we use 'that' to preserve the context
    this.cities.forEach(function (city) {
      console.log(`${that.name} has lived in ${city}`) //expected output
    }) 
  }
}
user2.printPlacesLived();

// with arrow functions there's no need for any workaround
const user3 = {
  name: 'Rocio',
  cities: ['Bogotá', 'Valencia', 'Caracas', 'Chicago', 'Melbourne'],
  printPlacesLived: function () {
    //arrow functions no longer bind their own 'this' value
    //instead they just use the 'this' value of the context they were created in
    //uses its parent's 'this' value
    this.cities.forEach((city) => {
      console.log(`${this.name} has lived in ${city}`) //expected output
    })
  }
}
user3.printPlacesLived();

//an arrow function wouldn't work here because it will try to use the parent's this which is undefined
...
printPlacesLived: () => {
  this.cities.forEach((city) => {
    console.log(`${this.name} has lived in ${city}`)
  })
}
...
//we can use the new syntax though to make it clearer
...
const user4 = {
  name: 'Rocio',
  cities: ['Bogotá', 'Valencia', 'Caracas', 'Chicago', 'Melbourne'],
  printPlacesLived() { //using new es6 syntax
    this.cities.forEach((city) => {
      console.log(`${this.name} has lived in ${city}`) //expected output
    })
  }
}
user4.printPlacesLived();
...
```

# map function
- **map** is an array method extensively used with JSX
- Like the ```forEach``` method, ```map``` gets called with a single function, this function gets called one time for every item in the array and we have access to that item via the first argument
- ```forEach``` lets you do something with each element whereas with ```map``` we can actually transform each item
- We get a new array back with the transformed items 
- The original array doesn't get changed
  ```javascript
  const user5 = {
    name: 'Rocio',
    cities: ['Miami', 'Chicago', 'Buenos Aires', 'Mexico City', 'Sydney', 'Brisbane'],
    printPlacesLived() {
      const cityMessages = this.cities.map((city) => {
        return `${this.name} has visited ${city}`
      })
      return cityMessages
    }
  }
  console.log(user5.printPlacesLived());
  //output: ["Rocio has visited Miami", "Rocio has visited Chicago", "Rocio has visited Buenos Aires", 
  //"Rocio has visited Mexico City", "Rocio has visited Sydney", "Rocio has visited Brisbane"]
  ```
- Simplified syntax
  ```javascript
  const user6 = {
    name: 'Rocio',
    cities: ['Miami', 'Chicago', 'Buenos Aires', 'Mexico City', 'Sydney', 'Brisbane'],
    printPlacesLived() {
      return this.cities.map((city) => `${this.name} has visited ${city}`)
    }
  }
  console.log(user6.printPlacesLived());
  ```
- map challenge
  ```javascript
  // challenge
  const multiplier = {
    numbers: [8, 9, 5],
    multiplyBy: 2,
    multiply() {
      //the expression after => is implicitly returned
      return this.numbers.map((number) => number * this.multiplyBy)
    }
  }
  console.log(multiplier.multiply()) //output: [16, 18, 10]
  ```








<hr>  
  -
  -
  -
  -
  -
  -
<hr>

