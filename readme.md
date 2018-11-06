> markdown: https://guides.github.com/features/mastering-markdown/  
> Preview: Chrome extension and QuickLook / space key on the file 

React Course - Udemy - Andrew Mead
============================================
<!-- TOC -->

- [Setting up environment](#setting-up-environment)
- [Hello React](#hello-react)
- [JSX Basics](#jsx-basics)
- [JSX Comments](#jsx-comments)
- [JSX Expressions](#jsx-expressions)
- [JSX Conditional Rendering](#jsx-conditional-rendering)
  - [If statement](#if-statement)
  - [Ternary Operator](#ternary-operator)
  - [Logical and operator](#logical-and-operator)
  - [All together](#all-together)
- [Var vs. let/const](#var-vs-letconst)
- [Functions vs. Arrow functions - Syntax](#functions-vs-arrow-functions---syntax)
- [Functions vs. Arrow functions - Deep dive](#functions-vs-arrow-functions---deep-dive)
  - [arguments object](#arguments-object)
  - [this object](#this-object)
- [map function](#map-function)
- [Events and attributes](#events-and-attributes)
  - [Attributes](#attributes)
  - [Events](#events)
- [Manual Data Binding](#manual-data-binding)
- [Arrays](#arrays)
- [Forms, Inputs, Arrays and Random numbers](#forms-inputs-arrays-and-random-numbers)
- [React Virtual DOM readings](#react-virtual-dom-readings)
- [ES6 Classes](#es6-classes)
- [Subclasses](#subclasses)
- [React Components](#react-components)
  - [Nesting Components](#nesting-components)
  - [props](#props)
  - [Event handling](#event-handling)
  - [Method Binding](#method-binding)
- [State](#state)
  - [Setting up state](#setting-up-state)
    - [Example](#example)
- [Upstream communication](#upstream-communication)
- [Pass data upstream](#pass-data-upstream)
- [More than one state](#more-than-one-state)
- [Summary props vs state](#summary-props-vs-state)
  - [State](#state-1)
- [Stateless Functional Components](#stateless-functional-components)
  - [Comparison between Class based and Functional components - CC vs FC](#comparison-between-class-based-and-functional-components---cc-vs-fc)
- [Default props](#default-props)
- [Return an object from an arrow function using shorthand syntax](#return-an-object-from-an-arrow-function-using-shorthand-syntax)
- [Remove individual option](#remove-individual-option)
- [Lifecycle methods](#lifecycle-methods)
  - [componentDidMount](#componentdidmount)
  - [componentDidUpdate](#componentdidupdate)
  - [componentWillUnmount](#componentwillunmount)
- [LocalStorage](#localstorage)

<!-- /TOC -->

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

# JSX Comments
  ```javascript
  {/* comments in JSX */}
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

# Events and attributes

## Attributes
- Adding attributes to HTML elements in JSX is as easy as adding them to an a regular html element. However, some attributes such as ```class``` have been renamed for JSX. In this particular case ```class``` is a reserved word to create classes in javascript.
- Similar thing happens with some other attributes: https://reactjs.org/docs/dom-elements.html. Many of them have also been renamed to camelCase.
  ```javascript
  let count = 0
  const templateTwo = (
    <div>
      <h1>Count: {count}</h1>
      <button id="my_id" className="button">+1</button>
    </div>
  );
  const appRoot = document.getElementById('app');
  ReactDOM.render(templateTwo, appRoot);
  ```
- In the end of the day ```templateTwo``` is just an object with ```type: "div"``` (root) and sub-elements are inside ```props.children```. In turn, each sub-element has the same structure: a type and its children (as nodes)
![Template Object](./img/template_object.png)
- An attribute can be set equal to a javascript expression, whatever results from the expression will be the value for the attribute.
  ```javascript
  let count = 0
  const someId = 'myidhere'
  const templateTwo = (
    <div>
      <h1>Count: {count}</h1>
      <button id={someId} className="button">+1</button>
    </div>
  );
  ```
  
## Events
- Expressions as the value for attributes are very useful for event handling.
  ```javascript
  let count = 0
  const addOne = () => {
    console.log('addOne')
  }
  const templateTwo = (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={addOne}>+1</button>
    </div>
  );
  const appRoot = document.getElementById('app');
  ReactDOM.render(templateTwo, appRoot);
  ```
- We can also define the even handler inline. Better for short functions as it gets difficult to read and maintain.
  ```javascript
  let count = 0
  const templateTwo = (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => {
        console.log('adding one')
        }}>+1
      </button>
    </div>
  );
  const appRoot = document.getElementById('app');
  ReactDOM.render(templateTwo, appRoot);
  ```

# Manual Data Binding
- JSX does not have built-in data binding
- Trying to update the counter in the previous example using the same code will not have effect in the view (HTML)
- The expression ```const templateTwo = ...``` runs before anything is rendered to the screen. ```count``` is zero and the buttons haven't even been registered with the event handler functions. The actual render happens when we call ```ReactDOM.render()```
  ```javascript
  let count = 0
  const addOne = () => {
    count++
    console.log('addOne', count)
  }
  const templateTwo = (
    <div>
      <h1>Count: {count}</h1> // this will always be zero
      <button onClick={addOne}>+1</button>
    </div>
  );
  const appRoot = document.getElementById('app');
  ReactDOM.render(templateTwo, appRoot);
  ```
- As a workaround we can move the template and rendering code to a function and call that function once when the page loads and then any time we change the data (**manual binding**). Later we'll use react components to do this.
  ```javascript
  let count = 0
  const addOne = () => {
    count++
    renderCounterApp() //re-render when data changes
  }
  const appRoot = document.getElementById('app')
  const renderCounterApp = () => {
    const templateTwo = (
      <div>
        <h1>Count: {count}</h1>
        <button onClick={addOne}>+1</button>
      </div>
    );
    ReactDOM.render(templateTwo, appRoot)
  }
  renderCounterApp() //initialize the App
  ```
- Rendering UI is very expensive but React makes the re-rendering super efficient. React uses virtual DOM algorithms to determine the portions on the screen that need to be updated and changes only that small little section. No even changing whole tags (such as ```<h1>```) but the text node that contains the counter in this example. What comes back from ```React.createElement()``` is just an object that represents our entire JSX tree. React uses algorithms to compare two objects which is way less expensive that re-painting pixels to the browser.

# Arrays
- JSX support arrays by default (as well as strings and numbers. Does not support objects and ignore booleans, null and undefined)
- When JSX sees an array it breaks each element out into their own expressions and shows them side by side
  ```javascript
  { [99, 98, 'Rocio'] } is equivalent to { {99}{98}{'Rocio'} } 
  { [null, undefined, true] } is equivalent to { {null}{undefined}{true} } //none will render
  ```
- We know we can also render JSX inside of JSX e.g ```{<p>My Paragraph</p>}``` and that will show up to the screen. In turn, we can render an array of JSX inside JSX. Most of the time we will not have arrays of numbers/strings to render to the screen but instead arrays of JSX expressions we want to render to the screen, for example an array of paragraphs or list items ```<li>```
  ```javascript
  { [<p>a</p>, <p>b</p>, <p>c</p>] }
  //will render as:
  //a
  //b
  //c
  ```
- With JSX inside arrays, we get an error: ```Warning: Each child in an array or iterator should have a unique "key" prop.``` and this is to help JSX to know when to re-render small parts of the application.
  ```javascript
  { [<p key="1">a</p>, <p key="2">b</p>, <p key="3">c</p>] }
  ```
- To display an array of elements as JSX code we use the map function
  ```javascript
  const numbers = [55, 101, 1000]
  
  //inside render function
  {
    numbers.map((number) => {
      return <p key={number}>Number: {number}</p>
    })
  }
  ```
  
# Forms, Inputs, Arrays and Random numbers
- Example below takes the input and add it to the options array. Also allow to remove all options and re-renders every time the data changes. Allows to pick a random option. Handle events onSubmit and onClick and shows conditional attribute disabled. 
- For a list of events: React Dom Events https://reactjs.org/docs/events.html
- Math.random() generates a number between 0 and 0.999999 [0,1). It can be zero but not 1.
  ```javascript
  const app = {
    title: 'Indecision App',
    subtitle: 'Are you feeling lucky?',
    options: []
  };

  const onFormSubmit = (e) => {
    e.preventDefault() //avoid re-rendering the whole page
    const option = e.target.elements.option.value //elements.option is by name
    if(option) { //no empty
      app.options.push(option)
      e.target.elements.option.value = '' //clear up the input field
      renderApp()
    }
  }

  const onRemoveAll = () => {
    app.options = [] //assign an empty array
    renderApp()
  }

  const onMakeDecision = () => {
    const randomNun = Math.floor(Math.random() * app.options.length)
    const option = app.options[randomNun]
    alert(option)
  }

  const appRoot = document.getElementById('app')

  const renderApp = () => {
    const template = (
      <div>
        <h1>{app.title}</h1>
        {app.subtitle && <p>{app.subtitle}</p>}
        <p>{app.options.length > 0 ? 'Here are your options' : 'No options'}</p>
        <button disabled={app.options.length === 0} onClick={onMakeDecision}>What should I do?</button> 
        {/* button is disabled when array is empty */}
        <button onClick={onRemoveAll}>Remove All</button> 
        {/*nothing to submit so no form associated*/}
        <ol>
          {app.options.map((option) => <li key={option}>{option}</li>)} {/*key is important*/}
        </ol>
        <form onSubmit={onFormSubmit}>
          <input type="text" name="option" />
          <button>Add Option</button>
        </form>
      </div>
    );
    ReactDOM.render(template, appRoot)
  }

  renderApp() //render App for the first time
  ```
  
# React Virtual DOM readings
- https://reactjs.org/docs/reconciliation.html
- https://medium.com/@gethylgeorge/how-virtual-dom-and-diffing-works-in-react-6fc805f9f84e
- https://www.youtube.com/watch?v=mLMfx8BEt8g

# ES6 Classes
- ```class``` was always a reserved word but until ES6 it wasn't useful
- By convention classes names start with uppercase ```class Person```
- Use the ```new``` keyword to create instances of the class ```const me = new Person()```
- Use the ```constructor``` function to set individual attributes. The constructor gets implicitly called when we crete a new instance of a class.
  ```javascript
  class Person {
    constructor(name) {
      this.name = name // customize individual instance of Person
    }
  }

  const me = new Person('Rocio')
  console.log(me)
  ```
- We don't have to pass all values to create an instance, it that case name will be undefined. ```const other = new Person()```
- We can also provide defaults in case the data is not specified. In the past it was done with a workaround using the logical OR operator
  ```javascript
  class Person {
    constructor(name) {
      this.name = name || 'Anonymous' //use the name if it exists otherwise use 'Anonymous'
    }
  }
  const other = new Person()
  console.log(other)
  ```
- With ES6 we have access to **function defaults**, these can be used anywhere we define an argument list 
  ```javascript
  class Person {
    constructor(name = 'Anonymous') { //function defaults
      this.name = name
    }
  }
  const other = new Person()
  console.log(other)
  ```
- Constructors take care of what makes each person unique. Now, the things each person shares are modelled using **methods** which allow us to reuse code across every instance of a Person. A method need to be explicitly called for it to run.
- :warning: Adding a comma between the constructor and methods or between methods is not allowed
- Methods are available to us on the individual instances of our class
  ```javascript
  class Person {
    constructor(name = 'Anonymous') {
      this.name = name
    } //no comma here to separate constructor from methods
    getGreeting() {
      return `Hi. I am ${this.name}!` //ES6 template strings
    }
  }
  const me = new Person('Rocio')
  console.log(me.getGreeting())
  ```

# Subclasses
- We can extend the class Person and that's going to give us all the Person's functionality and it's also going to allow us to override any of the functionality we might want to change. ES6 does not support multiple inheritance.
  ```javascript
  class Student extends Person {
    constructor(name, age, major) {
      super(name, age)  //calling the parent constructor
      this.major = major
    }
  }
  const me = new Student('Rocio', 38, 'Computer Science')
  const other = new Student()
  ```
- We can add new methods specific to the subclass
  ```javascript
  class Student extends Person {
    constructor(name, age, major) {
      super(name, age)  //calling the parent constructor
      this.major = major
    }
    hasMajor() {
      console.log(this.major) //string or undefined
      console.log(!this.major) //false if there's a string, true otherwise
      return !!this.major //returns boolean true if there's a string, false otherwise (instead of truthy, falsy)
    }
  }
  const me = new Student('Rocio', 38, 'Computer Science')
  console.log(me.hasMajor()) //true
  const other = new Student()
  console.log(other.hasMajor()) //false
  ```
- We can also override a method provided by the parent class
  ```javascript
  class Person {
    constructor(name = 'Anonymous', age = 0) {
      this.name = name
      this.age = age
    }
    getGreeting() {
      return `Hi. I am ${this.name}!`
    }
    getDescription() {
      return `${this.name} is ${this.age} year(s) old.`
    }
  }

  class Student extends Person {
    constructor(name, age, major) {
      super(name, age)  //calling the parent constructor
      this.major = major
    }
    hasMajor() {
      return !!this.major //returns boolean true if there's a string, false otherwise
    }
    getDescription() {
      let description = super.getDescription() //calling parent's method
      if(this.hasMajor()) {
        description += ` Their major is ${this.major}.`
      }
      return description
    }
  }

  const me = new Student('Rocio', 38, 'Computer Science') 
  console.log(me.getDescription()) //Rocio is 38 year(s) old. Their major is Computer Science.

  const other = new Student()
  console.log(other.getDescription()) //Anonymous is 0 year(s) old.
  ```
  
# React Components
- Components allow us to break up our App into small reusable chunks (think of it as UI sections)
- Each little component has its own set of JSX that React renders to the screen, it can handle events for those JSX elements and allow us to create these little self-contained units 
- A React Components is an ES6 class that extends from ```React.Component``` (Global we get when adding React to our project)
- React Components require the method ```render()``` to be defined. From this method we return the JSX for our Component
- To render a Component we still use ```ReactDOM.render``` passing the JSX template we want to render and the place inside the HTML where we want it to be shown
- Component classes must start with uppercase, React enforces this and it's a way to differentiate between regular HTML tags such as ```<h1></h1>``` and Components such as ```<Header />```. Using uppercase, the compiled code looks like ```React.createElement(Header, null)``` while with lowercase, it will try to render a 'header' HTML tag ```React.createElement('header', null),```. It's important to follow the convention, otherwise the Component will not render.
- React Component Example
  ```javascript
  class Header extends React.Component {
    render() {
      return (
        <div>
          <h1>Indecision</h1>
          <h2>Put your life in the hands of a computer</h2>
        </div>
      );
    }
  }
  const jsx = (
    <div>
      <Header />
    </div>
  )
  ReactDOM.render(jsx, document.getElementById('app'))
  ```

## Nesting Components
- React Components can render another components allowing us to nest components.
  ```javascript
  class Option extends React.Component {
    render() {
      return (
        <div>
          <p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Option component here</p>
        </div>
      );
    }
  }
  class Options extends React.Component {
    render() {
      return (
        <div>
          <p>Options component here</p>
          <Option /> {/* nesting */}
          <Option /> {/* reusing */}
        </div>
      );
    }
  }
  ```

## props
- Allow components to communicate with one another. With props we can establish one way communication between a Parent and a Child components
- Props are read-only
- Setting props is similar to setting key-value pairs in HTML attributes
- This is a way to make the Component more flexible and pass data as we need
- props names can be anything eg ```<Header title="Test Title"```
- We use ```this.props``` to get access to data passed down to the Component
  ```javascript
  class IndecisionApp extends React.Component {
    render() {
      return (
        <div>
          <Header title="Test Title"/>
        </div>
      );
    }
  }

  class Header extends React.Component {
    render() {
      console.log(this.props)  {/* {title: "Test Title"} */}
      return (
        <div>
          <h1>{this.props.title}</h1>
          <h2>Put your life in the hands of a computer</h2>
        </div>
      );
    }
  }
  ```
- We can also set variables with data we want to pass in. Any Javascript code must be before the return statement or inside a JSX expression
  ```javascript
  class IndecisionApp extends React.Component {
    render() {
      const title = 'Indecision'
      const subtitle = 'Put your life in the hands of a computer'
      return (
        <div>
          <Header title={title} subtitle={subtitle}/>
        </div>
      );
    }
  }

  class Header extends React.Component {
    render() {
      return (
        <div>
          <h1>{this.props.title}</h1>
          <h2>{this.props.subtitle}</h2>
        </div>
      );
    }
  }
  ```
- Collection of elements can also get passed between components
  ```javascript
  class IndecisionApp extends React.Component {
    render() {
      const options = ['thing one', 'thing two', 'thing four']
      return (
        <div>
          <Options options={options} />
        </div>
      );
    }
  }

  class Options extends React.Component {
    render() {
      return (
        <div>
          {/* map returns a new array with all elements changed to be now <p> tags whose key and value are the option text*/}
          {
            this.props.options.map((option) => <p key={option}>{option}</p>)
          }
        </div>
      );
    }
  }
  ```
- We can keep on passing data as props from component parent to children. From ```<IndecisionApp />``` to ```<Options />``` to ```<Option />```
  ```javascript
  class IndecisionApp extends React.Component {
    render() {
      const options = ['thing one', 'thing two', 'thing four']
      return (
        <div>
          {/* <IndecisionApp /> passes options array to <Options /> */}
          <Options options={options} /> 
        </div>
      );
    }
  }

  class Options extends React.Component {
    render() {
      return (
        <div>
          {
            {/* <Options /> passes each optionText to <Option /> */}
            this.props.options.map((option) => <Option key={option} optionText={option} />)
          }
        </div>
      );
    }
  }

  class Option extends React.Component {
    render() {
      return (
        <div>
          {/* <Option /> uses the data to render the value passed from parent */}
          {this.props.optionText}
        </div>
      );
    }
  }
  ```

## Event handling
- We use methods inside each class to handle events associated to html elements rendered by such component. In this case onClick
  ```javascript
  class Action extends React.Component {
    handlePick() {
      alert('handlePick')
    }
    render() {
      return (
        <div>
          <button onClick={this.handlePick}>What should I do?</button>
        </div>
      );
    }
  }
  ```
- Submit Form
  ```javascript
  class AddOption extends React.Component {
    handleAddOption(e) { {/* gets called with the event */}
      e.preventDefault() {/* prevents default browser behavior - submit */}
      const option = e.target.elements.option.value.trim() {/* option here is the name of the input */}
      if (option) {
        alert(option)
      }
    }
    render() {
      return (
        <div>
          <form onSubmit={this.handleAddOption}>
            <input type="text" name="option" />
            <button>Add Option</button>
          </form>
        </div>
        
      );
    }
  }
  ```

## Method Binding
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind
- Javascript losses the ```this``` binding in certain situations
- We know this works
  ```javascript
  const obj = {
    name: 'Rocio',
    getName() {
      return this.name
    }
  }
  console.log(obj.getName()) //prints Rocio
  ```
- But this code breaks the binding. We can't get a reference to getName method. They both run the same code, the problem is that the context is different. ```obj.getName()``` is in the context of an object, so we have access to that object and the ```this``` binding. But when we break it out to a function we lose that context, the context does not get transferred. This is now a regular function and regular functions have undefined for ```this``` by default. This is not specific for React but instead all Vanilla Javascript, JQuery etc have this issue.
  ```javascript
  const getName = obj.getName //TypeError: undefined is not an object (evaluating 'this.name')
  console.log(getName())
  ```
- What we need to do is to set the ```this``` binding in certain situations. For this we can use the ```bind``` method available on functions in Javascript. In the first argument we can pass the ```this``` context
  ```javascript
  const obj = {
    name: 'Rocio',
    getName() {
      return this.name
    }
  }
  const getName = obj.getName.bind(obj) //now it's bind 
  console.log(getName()) //prints Rocio
  ```
- We can actually pass anything we want as context
  ```javascript
  const getName = obj.getName.bind({ name: 'Rossy44'}) //changes the context
  console.log(getName()) // prints Rossyp44
  ```
- Event handling is one of those situations where Javascript loses the binding to ```this```
  ```javascript
  class Options extends React.Component {
    handleRemoveAll() {
      console.log(this.props.options) //TypeError: undefined is not an object (evaluating 'this.props')
      alert('handleRemoveAll')
    }
    render() {
      return (
        <div>
          <button onClick={this.handleRemoveAll}>Remove All</button>
          {
            this.props.options.map((option) => <Option key={option} optionText={option} />)
          }
        </div>
      );
    }
  }
  ```
- By doing this, we're doing ```handleRemoveAll()``` to have the same binding as ```render()``` does
  ```javascript
  class Options extends React.Component {
    handleRemoveAll() {
      console.log(this.props.options)
      alert('handleRemoveAll')
    }
    render() { //render is not an event handler so its binding is OK
      return (
        <div>
          <button onClick={this.handleRemoveAll.bind(this)}>Remove All</button>
          {this.props.options.map((option) => <Option key={option} optionText={option} />)}
        </div>
      );
    }
  }
  ```
- This technique works but it's a little inefficient, it requires us to rerun ```bind``` anytime the component re-renders and because components re-render often this can get a bit expensive
- A better approach is to override the constructor function for the React Component
  ```javascript
  class Options extends React.Component {
    constructor(props) { //gets called with props
      super(props) //important to do this
      this.handleRemoveAll = this.handleRemoveAll.bind(this) 
      //now the context is correct and we only have to do this once
    }
    handleRemoveAll() {
      console.log(this.props.options)
      alert('handleRemoveAll')
    }
    render() {
      return (
        <div>
          <button onClick={this.handleRemoveAll}>Remove All</button>
          {this.props.options.map((option) => <Option key={option} optionText={option} />)}
        </div>
      );
    }
  }
  ```

# State
- Component state allows our components to manage some data (object with various key-value pairs) and when that data changes the component will automatically re-render to reflect those changes 
- Previously (jsx-indecision) we manually called a render method every time we needed to update something on the screen. With component state all we have to do is manipulate the 
data and the component is gonna take care of re-rendering itself

## Setting up state
1. Setup default state object (object with variables in zero, empty arrays etc)
2. Component rendered with default state values (*)
3. Change state based on event (button click, network request ready etc)
4. Component re-rendered using new state values (*)
5. Start again at 3

(*) behind the scenes

### Example
- Step 1
  ```javascript
  class Counter extends React.Component {
    constructor(props) {
      super(props)
      //step1 - setting up default state object
      this.state = {
        count: 0
      }
    }
    ...
  }
  ```
- Step 2
  ```javascript
  render() {
    return (
      <div>
        {/* step2 - component rendered with default state values */}
        <h1>Count: {this.state.count}</h1>
        <button onClick={this.handleAddOne}>+1</button>
        <button onClick={this.handleMinusOne}>-1</button>
        <button onClick={this.handleReset}>reset</button>
      </div>
    );
  }
  ```
- Step 3
  ```javascript
  handleAddOne() {
    //step3 - change state base on event
    //we can't just update the this.state object instead we call this.setState method
    //first parameter is a function that returns the new state and we get access to the previous state
    this.setState((prevState) => {
      return {
        count: prevState.count + 1
      }
    })
  }
  ```
- Step 4 happens automatically after setting the new state
- When setting the new state we provide just the ones we need to change, the rest remain the same
- We might not need the previous state and that's also fine
  ```javascript
  handleReset() {
    //no previous state needed
    this.setState(() => {
      return {
        count: 0
      }
    })
  }
  ```
- React allows to pass an object to setState instead of a function and it might bring issues specially when the previous state is needed. There will be inconsistency in the result because setState is asynchronous. Even if we don't need the previousState the recommended way is to pass a function as React might deprecate the alternative.
- Complete Counter example
  ```javascript
  class Counter extends React.Component {
    constructor(props) {
      super(props)
      //assign to themselves but rebinding the context
      //bound to the correct component instance when the callback eventually fires
      this.handleAddOne = this.handleAddOne.bind(this)
      this.handleMinusOne = this.handleMinusOne.bind(this)
      this.handleReset = this.handleReset.bind(this)
      //step1 - setting up default state object
      this.state = {
        count: 0
      }
    }
    handleAddOne() {
      //step3 - change state base on event
      //we can't just update the this.state object instead we call this.setState method
      //first parameter is a function that returns the new state and we get access to the previous state
      this.setState((prevState) => {
        return {
          count: prevState.count + 1
        }
      })
    }
    handleMinusOne() {
      this.setState((prevState) => {
        return {
          count: prevState.count - 1
        }
      })
    }
    handleReset() {
      //no previous state needed
      this.setState(() => {
        return {
          count: 0
        }
      })
    }
    render() {
      return (
        <div>
          {/* step2 - component rendered with default state values */}
          <h1>Count: {this.state.count}</h1>
          <button onClick={this.handleAddOne}>+1</button>
          <button onClick={this.handleMinusOne}>-1</button>
          <button onClick={this.handleReset}>reset</button>
        </div>
      );
    }
  }
  ReactDOM.render(<Counter />, document.getElementById('app'))
  ```

# Upstream communication
- props enables one way communication from parent components to a child, but children can't pass props upstream. To solve this problem we're gonna pass functions in as props. So, the parents defines the function and its behavior, and the children can use it to execute the task.
- Example. **Remove All** button (in <Options /> component) need to wipe up one piece of state (options array)
  - Define function and behavior in the parent and pass the function to the Child through props
    ```javascript
    class IndecisionApp extends React.Component {
      constructor(props) {
        super(props)
        this.handleDeleteOptions = this.handleDeleteOptions.bind(this)
        this.state = {
          options: ['thing one', 'thing two', 'thing three']
        }
      }
      //Parent define function and behavior
      handleDeleteOptions() {
        this.setState(() => {
          return {
            options: []
          }
        })
      }
      render() {
        return (
          <div>
            <Options 
              options={this.state.options} 
              /* { Passing the Parent's function to the Child through props } */
              handleDeleteOptions={this.handleDeleteOptions} 
            />
          </div>
        );
      }
    }
    ```
  - Wire up the Reference to the Parent's method in the Child so that when the event is fired the method is called
    ```javascript
    class Options extends React.Component {
      render() {
        return (
          <div>
            /* { Doing Something in the Parent } */
            <button onClick={this.props.handleDeleteOptions}>Remove All</button>
            {this.props.options.map((option) => <Option key={option} optionText={option} />)}
          </div>
        );
      }
    }
    ```
    
# Pass data upstream
- We still pass down a function, but we define a parameter so that the child can send data to the parents through the parameters list
  ```javascript
  //In the parent
  class IndecisionApp extends React.Component {
    constructor(props) {
      super(props)
      this.handleAddOption = this.handleAddOption.bind(this)
      this.state = {
        options: []
      }
    }
    handleAddOption(option) {
      this.setState((prevState) => {    
        return {
          //we don't want to manipulate prevState or state that's why we don't use push
          //concat return a new array without manipulating the old ones
          options: prevState.options.concat(option) // we can pass an array or an individual value
        } 
      })
    }
    ...
  }
  ```
  ```javascript
  //In the child
  class AddOption extends React.Component {
    constructor(props) {
      super(props)
      this.handleAddOption = this.handleAddOption.bind(this)
    }
    handleAddOption(e) {
      e.preventDefault()
      const option = e.target.elements.option.value.trim()
      if (option) {
        this.props.handleAddOption(option) //calling parent method and passing option upstream
      }
    }
    render() {
      return (
        <div>
          <form onSubmit={this.handleAddOption}>
            <input type="text" name="option" />
            <button>Add Option</button>
          </form>
        </div>
        
      );
    }
  }
  ```

# More than one state
- Each component (class) can have its own state. This example uses state to track errors
  ```javascript
  //In the parent
  class IndecisionApp extends React.Component {
    constructor(props) {
      super(props)
      this.handleAddOption = this.handleAddOption.bind(this)
      this.state = {
        options: []
      }
    }
    handleAddOption(option) { //will return undefined if all good otherwise a message
      if (!option) {
        return 'Enter valid value to add item'
      } else if(this.state.options.indexOf(option) > -1) {
        return 'This option already exists'
      }
      this.setState((prevState) => {    
        return {
          //we don't want to manipulate prevState or state that's why we don't use push
          //concat return a new array without manipulating the old ones
          options: prevState.options.concat(option) // we can pass an array or an individual value
        } 
      })
    }
    render() {
      const title = 'Indecision'
      const subtitle = 'Put your life in the hands of a computer'
      return (
        <div>
          <AddOption handleAddOption={this.handleAddOption} />
        </div>
      );
    }
  }
  ```
  ```javascript
  //In the child
  class AddOption extends React.Component {
    constructor(props) {
      super(props)
      this.handleAddOption = this.handleAddOption.bind(this)
      this.state = {
        error: undefined
      }
    }
    handleAddOption(e) {
      e.preventDefault()
      const option = e.target.elements.option.value.trim()
      const error = this.props.handleAddOption(option) //calls parent's method
      this.setState(() => {
        //same as return { error: error } but ES6 provides shorthand
        return { error }
      })
    }
    render() {
      return (
        <div>
          { /* if there's an error show it */ }
          {this.state.error && <p>{this.state.error}</p>} 
          <form onSubmit={this.handleAddOption}>
            <input type="text" name="option" />
            <button>Add Option</button>
          </form>
        </div>
        
      );
    }
  }
  ```
  
# Summary props vs state
![Props vs State](./img/props_vs_state.png)

## State
![State](./img/state.png)

# Stateless Functional Components
- Used for components that are very simple and do not need to keep state. Common for simple presentational components
- Functional components are functions instead of classes
- The inside of the function is very similar to the render method in a class component. Must return the JSX to present in the screen
- The usage of a Functional Component is exactly the same as a Class Component
  ```javascript
  const User = () => {
    return (
      <div>
        <p>Name: </p>
        <p>Age: </p>
      </div>
    );
  }
  ReactDOM.render(<User />, document.getElementById('app'))
  ```
- Functional components do not have access to ```this```
- Functional components do not have state but they still can get access to ```props``` passed in. props are passed as the first argument to the function
  ```javascript
  const User = (props) => {
    return (
      <div>
        <p>Name: {props.name}</p>   {/* in Class components we use this.props.name */}
        <p>Age: {props.age}</p>
      </div>
    );
  }
  {/* note that age is passed in as an expression */}
  ReactDOM.render(<User name="rocio" age={38} />, document.getElementById('app'))
  ```
  
## Comparison between Class based and Functional components - CC vs FC
- FC are faster than CC. We should use FC anywhere we can
- FC are a bit easier to read and write
- FC are easier to test
- CC have lifecycle methods that fire at various times in a given component's life. FC don't have this.

# Default props
- Allow us to define defaults for props on both Class based components as well as Functional components. In case a prop is no passed in to the component, React will use the default value instead
- Default props is an object assigned to the Component's defaultProps
  ```javascript
  Header.defaultProps = {
    title: 'Some Default!'
  }
  ```
- If the component is used with providing a ```title```, the default value is used
  ```javascript
  <Header subtitle={subtitle} />
  ```

# Return an object from an arrow function using shorthand syntax
- We need to wrap the function body with parenthesis 
- Object uses {} but the arrow function will interpret the braces as the function body
  ```javascript
  const num = () => 12 + 2    //return a number
  const num = () => [1, 2, 3] //return an array
  const num = () => ({})      //return empty object
  const num = () => {}        //arrow function that returns nothing - undefined
  this.setState(() => ({ options: [] })) //set state as an object that contains the property options as an empty array
  ```

# Remove individual option
- A prop can be passed several levels down
- The onClick in Option has to be managed further because handleDeleteOption is expecting the text of the option to remove not the event
  ```javascript
  //in IndecisionApp
  handleDeleteOption(optionToRemove) {
    this.setState((prevState) => ({
      options: prevState.options.filter((option) => optionToRemove !== option)
    }))
  }
  <Options 
    handleDeleteOption={this.handleDeleteOption}
  />
  ...
  //in Options
  <Option 
    handleDeleteOption={props.handleDeleteOption}
  />
  ...
  //in Option
  const Option = (props) => {
    return (
      <div>
        {props.optionText}
        <button
          onClick={(e) => {
            props.handleDeleteOption(props.optionText)
          }}
        >
          remove
        </button>
      </div> 
    );
  }
  ```

# Lifecycle methods
https://reactjs.org/docs/react-component.html

## componentDidMount
- Fires when the component first get mounted to the DOM
- We never call this methods explicitly, React will call them
  
## componentDidUpdate
- Fires after the component updates. eg. state values change or prop values change
- Inside this method we have access to ```this.state``` and ```this.props``` which contains the new values
- We also have access to previous state and props as arguments in the method

## componentWillUnmount
- Fires just before the component goes away off the screen

# LocalStorage
- Data saved in the format key-value pairs
- localStorage is going to persist between page loads
- save data: ```localStorage.setItem('key', 'value')```
- get data: ```localStorage.getItem('key')```
- remove data: ```localStorage.removeItem('key')```
- clear ALL data: ```localStorage.clear()```
- localStorage only works with string data. If we pass something different like a number it will convert it to string and we need to parse it back using ```parseInt(num)``` or whatever data type we're working with. We can combine this with ```isNaN()``` to validate it's a number
- JSON is a string representation of a javascript object, we can use JSON to save/retrieve more complex data
- ```JSON.stringify({ age: 26 })``` takes a regular javascript object and returns the string representation
- ```JSON.parse(json)``` takes the string representation and returns a true javascript object
- localStorage example
  ```javascript
  componentDidMount() {
    try { //handling malformed JSON
      const json = localStorage.getItem('options')
      const options = JSON.parse(json)
      if (options) { //checking there's data
        this.setState(() => ({ options }))
      } 
    } catch (e) {
      // Do nothing at all
    }
  }
  componentDidUpdate(prevProps, prevState) {
    //componentDidUpdate fires even when data is the same
    //the if avoids updating when there's not needed
    if (prevState.options.length != this.state.options.length) {
      const json = JSON.stringify(this.state.options)
      localStorage.setItem('options', json)
    }
  }
  ```

<hr>
  -
  -
  -
  -
  -
  -
<hr>

