# React Workshop

## Getting Started

* Assuming you're using Sublime Text 3 (if you are not, please download it):
	*  download [Package Control](https://packagecontrol.io/installation)
	*  then using Package Control, download "Babel"

	
Now you have React/ES6 syntax highlighting for Sublime Text 3!

## Setup

Once you've downloaded this repo run the following commands from within this projects directory (_this instruction assumes you have node and npm already installed)_:

```
npm install
sudo npm install webpack-dev-server -g
```

Then to run the app:

```
webpack-dev-server --hot --inline
```

When visiting `http://localhost:8080/` in your browser you should see a "Hello World!" message. `webpack-dev-server` provides automatic reloads & updates.

## ES6 modules & JSX

JavaScript ES5 modules are an officially approved spec, however no browser has implemented support for modules. Importing a JavaScript ES6 module look like:

```
import React from 'react';
```

Despite the lack of support, the React community uses ES6 modules heavily through a transpiler called [Babel](https://babeljs.io/). Babel transpiles (more or less compiles) ES6 code to ES5 code, which all browsers support. This allows us to use all the goodness of ES6 without having to wait for browser support.

Likewise, React developers have created a `jsx` a superset of JavaScript that allows developers to add `html` inline to their JavaScript code:

```jsx
let App = () => {
  return (
    <h1>Hello World</h1>
  );
}
```

Again, browsers do not support `.jsx`, but Babel does.

The software that runs Babel (a plugin) is Webpack. Webpack also has other plugins like automate reload and other goodies (e.g. `webpack-dev-server`).

## Sourcemaps

One of the major kinks about using transpiled code is using Chrome's debugger. The transpiled code looks different then the code we wrote, has different line numbers and both of those aspects make using the debugger tricky. Thankfully we can use another technology that is widely supported by browsers called sourcemaps. Using an extra flag on Webpack we can tell it to generate a source map during the transpilation step. The source map literally provides the browser with a map between the source code that we wrote (ES6 + JSX) to the transpiled ES5 code. This allows us to "debug" the ES6/JSX code that we wrote and not the transpiled code that was generated by Webpack/Babel.

## React

Open up `index.html`

```jsx
import React from 'react';
import { render } from 'react-dom';

let App = () => {
  return (
    <div>
      <h1 style={{color: 'gray'}}>
        Hello World!
      </h1>
    </div>
  );
}

render(<App/>, document.getElementById('root'));
```

The first two lines import React and ReactDOM. ReactDOM is for interacting with the DOM (it was extracted out of React with the creation of ReactNative).

The function `App` is a React component - by convention we capitalize components. Note the mixture of JavaScript (ES6) and HTML - this mixture is JSX. Defining styles inline is also common in React - it makes the components more modular.

There are a number of ways to create React components, the methods above is the ["stateless" functional method](https://facebook.github.io/react/docs/reusable-components.html#stateless-functions). React developers on why you should make stateless components:

> In an ideal world, most of your components would be stateless functions because in the future we’ll also be able to make performance optimizations specific to these components by avoiding unnecessary checks and memory allocations. This is the recommended pattern, when possible.

The final line calles our new component `<App/>` and adds it to `<div id="root"></div>`, which is already on our `index.html` page.


## Create a new component

We're going to create a `CatPic` component and eventually allows us to resize the pic dynamically. First create a new file called `cat-pic.jsx` within the `src/` folder.

```jsx
import React from 'react';

let CatPic = () => {
  return (
    <div>
      <img 
        style={{width: 250 + 'px'}} 
        src="https://i.ytimg.com/vi/tntOCGkgt98/maxresdefault.jpg"
      />
    </div>
  );
}

export default CatPic;
```

Even though we don't use it explicitly we first we import `React`. Next we create a function called `CatPic` (this is our component). By React convention functions that are components are capitalized. 

### Using the component

Update `src/app.jsx` to import our new component and use it:

```
import React from 'react';
import { render } from 'react-dom';
import CatPic from './cat-pic.jsx';

let App = () => {
  return (
    <div>
      <h1 style={{color: 'gray'}}>
        Hello World!
      </h1>
      <CatPic />
    </div>
  );
}

render(<App/>, document.getElementById('root'));
```

In your browser you should now see a picture of a cat that's only 250 pixels wide.

## Create a button component 

Our above app is entirely static, let's add two buttons that allow us to increase and decrease the size of the cat pic. First create a new file: `src/change-pic-size.jsx` and create a new component called `ChangePicSize` that includes a `<div>` with two buttons, each with an `onClick` function attached. For now each button will just log to the console.

### Hint

* Don't forget to import React
* Don't forget to export your component
* onClick is a property of the button element `<button onClick={() => {}}>`
* `import` your componet in `app.jsx` and add the component after `<CatPic/>`

### Cheat

```jsx
import React from 'react';

let ChangePicSize = () => {
  return (
    <div>
      <button onClick={() => {
        console.log('increase size...');
      }}>+ Increase</button>
      <button onClick={() => {
        console.log('decrease size...');
      }}>- Decrease</button>
    </div>
  );
}

export default ChangePicSize;
```

## Connecting the button component to `app.jsx`

Add the following import to `app.jsx`:

```jsx
import ChangePicSize from './change-pic-size.jsx';
```

Then add `<ChangePicSize />` right below `<CatPic />`:

```jsx
  <CatPic />
  <ChangePicSize />
```

You should now see two buttons, that when clicked print to the console.
    

## Redux & One-way flow of data

Redux, a library created by a Facebook developer, facilitates one-way data flow simply and elegantly. In order to make our buttons do something we're going to integrate Redux with less than 20 lines of code!

How Redux works:

1. It has a single state object that stores your application's entire state. This might sound bad, but makes testing very easy.
2. It uses functions (Redux calls them "reducers") to modify the state object.
3. Events are emmitted by components of your app (e.g. button clicks), Redux picks up these events and the reducer handles them to modify the state.
4. The new state flows through all the components, React figures out what components need to update and which don't.

### Update the imports in `app.jsx`

```jsx
import React from 'react';
import { render } from 'react-dom';
import { createStore, combineReducers } from 'redux';
import { Provider } from 'react-redux';

import CatPic from './cat-pic.jsx';
import ChangePicSize from './change-pic-size.jsx';
```

We're importing two functions from redux, `createStore` and `combineReducers`. `combineReducers` accepts many reducers and returns a single reducer (as your app grows, so will the number of reducers). `createStore` accepts the reducer you've created and creates the app's store.

### Creating the store and reducer

After the `import` statements above add:

```jsx
const appReducers = combineReducers({
  picSize: function(){ return {}; }
});

let store = createStore(appReducers);
```

The reducer that we're calling `picSize` does nothing at the moment, it's just an empty function. Eventually it will be responsible for change the size of picture.

### Update the `render` call

Again, in `app.jsx` update the render to read:

```jsx
render(
  <Provider store={store}>
    <App/>
  </Provider>
, document.getElementById('root'));
```

Using `Provider` will help to automatically inject the app's state object into each component. At this point there should be no errors in your console - if there are please double check your code and the error!

## Creating Our Reducer Function

Create a new folder in `src` called `reducers`. In `reducers` create a file called `pic-size.jsx` - this will hold your reducer. The reducer is a function that takes two parameters, state and an action object. In our current reducer state will be `picSize`, a number. If `action.type` is `'INCREASE_SIZE'` then we want to return  `picSize + 10`. If it's `'DECREASE_SIZE'` then we `picSize - 10`. 

if `picSize` is undefined return `250`, the default value. If the action is not recognized return `picSize`.

Try and create the pic size reducer on your own, below is the cheat.

### Cheat

```jsx
const picSizeReducer = (picSize=250, action) => {
  switch(action.type){
    case 'INCREASE_SIZE':
      return picSize + 10
    case 'DECREASE_SIZE':
      return picSize - 10;
    default:
      return picSize;
  }
}

export default picSizeReducer;
```

In the above we're using ES6 default parameter (`picSize=250`) to deal with the scenario of picSize being undefined. 

### Connect our reducer to our app

In `app.jsx` import the new reducer:

```jsx
import picSizeReducer from './reducers/pic-size.jsx';
```

Then update the `appReducers` to use the new function:

```jsx
const appReducers = combineReducers({
  picSize: picSizeReducer
});

```

At this point your app should still be error free, but the buttons are still just print to the console.

## Connecting `ChangePicSize` to redux

Open `change-pic-size.jsx` and first add the following import:

```jsx
import { connect } from 'react-redux';
```

`connect` is the function that we'll use to _connect_ our component to redux (so it can send messages).

Right before the `export` statement add the following:

```jsx
ChangePicSize = connect()(ChangePicSize);
```
This line effectively creates a new version of ChangePicSize that is connected to redux and takes an object as a parameter that includes redux's dispatcher (What we'll use to send messages to update the state).

Finally, update `ChangePicSize` component to:

```jsx
let ChangePicSize = ({dispatch}) => {
  return (
    <div>
      <button onClick={() => {
        dispatch({
          type: 'INCREASE_SIZE'
        });
      }}>+ Increase</button>
      <button onClick={() => {
        dispatch({
          type: 'DECREASE_SIZE'
        });      
      }}>- Decrease</button>
    </div>
  );
}
```

`ChangePicSize` now takes an object as a parameter, one of its keys is called `dispatch`, which points to the dispatch function that allows us to trigger events. In the first onclick we send an `'INCREASE_SIZE'` message and the second we send a `'DECREASE_SIZE'`.

Our component now sends the message to redux, which has a reducer function to change the state. We now need to connect our `CatPic` component to redux.

## Connecting `CatPic` to redux

Open `cat-pic.jsx` and add the following import:

```jsx
import { connect } from 'react-redux';
```

Right above the `export` add the following code:

```jsx
const mapStateToProps = (state) => {
  return { picSize: state.picSize };
}

CatPic = connect(mapStateToProps)(CatPic);
```

The function `mapStateToProps` tells redux how to map the global state object to a `params` object that our component can use. 

Finally update `CatPic`:

```jsx
let CatPic = ({picSize}) => {
  return (
    <div>
      <img 
        style={{width: picSize + 'px'}} 
        src="https://i.ytimg.com/vi/tntOCGkgt98/maxresdefault.jpg"
      />
    </div>
  );
}
```

Now when you click on either button the size of the pictures should change!

## Extras

* [Redux documentation](http://redux.js.org/index.html)
* [30 short, getting started with redux videos](https://egghead.io/series/getting-started-with-redux)
* Defining components like we're doing above is not currently the most performant method, although Facebook developers promise that it will receive optimizations. Currently the most performant method to create components is: `class CatPic extends React.Component` (e.g. ES6 Classes). [More on performance](https://facebook.github.io/react/docs/advanced-performance.html)
* Putting everything in `app.jsx`, like we do above, will quickly get out of control. In larger apps I've seen all the reducers being importated into a different file (e.g. `reducers/index.jsx`), then importing that file.



## Bonus section

You might be wondering how you'd handle more than one picture. Specifically, you'd only want one pair of buttons to update the size of one picture - not all the pictures. Our current state `picSize` only stores a single value - the reducer `picSizeReducer` would have to be updated to accomodate an array of objects. In the array of objects, each object would contain a single `size`, an `id` and the images `src`. 

### Clues

* Create a "container" component `CatPicsContainer` that iterates over all the cat pics and for each one uses one `<CatPic ...>` and one `<ChangePicSize ...>`
* You'll need to pass the picId to `<ChangePicSize picId={catPic.id}>` so it can include the id when it dispatches the message.
* Reducers should have no "side-effects" and should not modify the state object, just *create* and new state object (you should consider the state object immutable).
* ES6 JavaScript arrays have a function called [`findIndex`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex) - it may be useful.
* To create a new array from parts of an existing array you can use the ES6 ["spread"](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Spread_operator) operator.
* You'll need to update `app.jsx`, `cat-pix.jsx` and `reducers/pic-size.jsx`.
* You'll need to create a new file `cat-pics-container.jsx`.
* You can "make-up" the id for each image.

### Cheat - CatPicsContainer

```jsx
import React from 'react';
import { connect } from 'react-redux';

import CatPic from './cat-pic.jsx';
import ChangePicSize from './change-pic-size.jsx';

// catPics would likely come from a server
let catPics = [
    { id: 1, size: 250, src: 'https://i.ytimg.com/vi/tntOCGkgt98/maxresdefault.jpg' },
    { id: 2, size: 250, src: 'http://www.medhatspca.ca/sites/default/files/news_photos/2014-Apr-15/node-147/cute-little-cat.jpg'}
  ];

let CatPicsContainer = ({catPics}) => {
  return (
    <div>
      {catPics.map((catPic) => {
        return <div key={catPic.id}>
          <CatPic src={catPic.src} size={catPic.size}/>
          <ChangePicSize picId={catPic.id}/>
        </div>
      })}
    </div>
  );
}

const mapStateToProps = (state) => {
  return { catPics: state.catPics };
}

CatPicsContainer = connect(mapStateToProps)(CatPicsContainer);

export { catPics, CatPicsContainer };
export default CatPicsContainer;
```

### Cheat - `app.jsx`

Make the following updates

```jsx
import { catPics, CatPicsContainer } from './cat-pics-container.jsx';

import picSizeReducer from './reducers/pic-size.jsx';

const appReducers = combineReducers({
  catPics: picSizeReducer
});

const initialState = {
  catPics: catPics
}
```

and

```jsx
let App = () => {
  return (
    <div>
      <h1 style={{color: 'gray'}}>
        Hello World!
      </h1>
      <CatPicsContainer />
    </div>
  );
}
```

### Cheat - `cat-pix.jsx`

```jsx
import React from 'react';

let CatPic = ({ size, src }) => {
  return (
    <div>
      <img 
        style={{width: size + 'px'}} 
        src={src}
      />
    </div>
  );
}

export default CatPic;
```

### Cheat - `reducers/pic-size.jsx`

Much of the complexity of solution below stems from the fact that `catPics` should not be mutated. The reducer should return either the exact same array (unchanged) or a new copy of `catPics`. To create a new copy we have to use the `...` (spread) operator. Alternatively you can use Facebook's [immutable.js library](https://facebook.github.io/immutable-js/) to create immutable data structures.

```jsx
import { catPics } from '../cat-pics-container.jsx';

const picSizeReducer = (pics=catPics, action) => {
  var picIndex = pics.findIndex((pic) => { return pic.id === action.picId });
  var pic = pics[picIndex];

  switch(action.type){
    case 'INCREASE_SIZE':
      pic = Object.assign({}, pic, {size: pic.size + 10});
      return [
        ...pics.slice(0, picIndex),
        pic,
        ...pics.slice(picIndex + 1, pics.length)
      ];
    case 'DECREASE_SIZE':
      pic = Object.assign({}, pic, {size: pic.size - 10});
      return [
        ...pics.slice(0, picIndex),
        pic,
        ...pics.slice(picIndex + 1, pics.length)
      ];
    default:
      return pics;
  }
}

export default picSizeReducer;
```

Or you can refactor above and do:

```jsx
import { catPics } from '../cat-pics-container.jsx';

let updateArray = (array, index, item) => {
  return [
    ...array.slice(0, index),
    item,
    ...array.slice(index + 1, array.length)
  ]
}

let updateObject = (obj, newKeyVals) => {
  return Object.assign({}, obj, newKeyVals);
}

const picSizeReducer = (pics=catPics, action) => {
  var picIndex = pics.findIndex((pic) => { return pic.id === action.picId });
  var pic = pics[picIndex];

  switch(action.type){
    case 'INCREASE_SIZE':
      pic = updateObject(pic, {size: pic.size + 10});
      return updateArray(pics, picIndex, pic); 
    case 'DECREASE_SIZE':
      pic = updateObject(pic, {size: pic.size - 10});
      return updateArray(pics, picIndex, pic);
    default:
      return pics;
  }
}

export default picSizeReducer;
```

Even better - put `updateArray` and `updateObject` into a utilities file and import them... or just use immutable.js.