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
- Each little component has its own set of JSX that it renders to the screen, it can handle events for those JSX elements and allow us to create these little self-contained units 
<hr>  
  -
  -
  -
  -
  -
  -
<hr>

