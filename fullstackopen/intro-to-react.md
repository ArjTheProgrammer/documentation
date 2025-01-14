# PART 1: Introduction to React

**NOTE**: Full set up of React project is still tentative for 
part 5 to part 7 (`testing react apps, advance state management,
React Router, custom hooks, styling apps with css, and webpack`) is 
still not finishedlike this

## Starting or making a React project

```
npm create vite@latest introdemo -- --template react
```

## Simplifying React main.jsx
```
import ReactDOM from 'react-dom/client'

import App from './App'

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

**App.jsx now looks like this**
```
const App = () => {
  return (
    <div>
      <p>Hello world</p>
    </div>
  )
}

export default App
```

## Making multiple react components

You can **create components** from the same file
and used it multiple times:

```
const Hello = () => {
  return (
    <div>
      <p>Hello world</p>
    </div>
  )
}

const App = () => {
  return (
    <div>
      <h1>Greetings</h1>
      <Hello />
      <Hello />
      <Hello />
    </div>
  )
}
```
---
**OR** you can make a **SEPARATE FILE** like this:

**Hello.jsx**
```
const Hello = () => {
  return (
    <div>
      <p>Hello world</p>
    </div>
  )
}

export default Hello;
```

**App.jsx**
```
import Hello from './Hello'

const App = () => {
  return (
    <div>
      <h1>Greetings</h1>
      <Hello />

      <Hello />
      <Hello />
    </div>
  )
}
```

## Props: passing data to the components
**Props** are data that is passed to the components

```
const Hello = (props) => {

  console.log(props)
  return (
    <div>
      <p>

        Hello {props.name}, you are {props.age} years old
      </p>
    </div>
  )
}

const App = () => {

  const name = 'Peter'
  const age = 10

  return (
    <div>
      <h1>Greetings</h1>

      <Hello name='Maya' age={26 + 10} />
      <Hello name={name} age={age} />
    </div>
  )
}
```

- the `Hello` function has a parameter of two props,
`props.name` and `props.age`. 

- Both is wrapped inside the tags with curly braces:
```
return (
    <div>
      <p>

        Hello {props.name}, you are {props.age} years old
      </p>
    </div>
  )
```

The `props` is defined in the App.jsx like this:
```
const App = () => {
  const name = 'Peter'
  const age = 10

  return (
    <div>
      <h1>Greetings</h1>
      <Hello name='Maya' age={26 + 10} />
      <Hello name={name} age={age} />
    </div>
  )
}
```

There can be an arbitrary number of props and 
their values can be "hard-coded" strings or the
results of JavaScript expressions. If the value
of the prop is achieved using JavaScript it must
be wrapped with curly braces.

**Destructure props like this**
```
const Hello = (props) => {
  const { name, age } = props
```

**OR** assign the values of the properties directly 
to variables by destructuring the props object that 
is passed to the component function as a parameter:

```
const Hello = ({ name, age }) => {
```

> **NOTE:** Do not render **OBJECTS** because it is not
as a react child

## Stateful component

The **STATE** object is where you store property 
values that belong to the component. When the state 
object changes, the component re-renders. ([w3schools, n.d.](https://www.w3schools.com/react/react_state.asp#:~:text=React%20components%20has%20a%20built,%2C%20the%20component%20re%2Drenders.))

Let's change the App.jsx like this:
```
import { useState } from 'react'

const App = () => {

  const [ counter, setCounter ] = useState(0)


  setTimeout(
    () => setCounter(counter + 1),
    1000
  )

  return (
    <div>{counter}</div>
  )
}

export default App
```

In the first row, the file imports the useState function:
```
import { useState } from 'react'
```

The function body that defines the component begins with the function call:

```
const [ counter, setCounter ] = useState(0)
```

The function call adds **state** to the component and renders it initialized with the value zero. The function returns an array that contains two items. We assign the items to the variables `counter` and `setCounter` by using the destructuring assignment syntax shown earlier.

The `counter` variable is assigned the initial value of state, which is zero. The variable `setCounter` is assigned a function that will be used to modify the state.

The application calls the `setTimeout` function and passes it two parameters: a function to increment the counter state and a timeout of one second:

```
setTimeout(
  () => setCounter(counter + 1),
  1000
)
```

When the state modifying function setCounter is called, React re-renders the component which means that the function body of the component function gets re-executed:

```
() => {
  const [ counter, setCounter ] = useState(0)

  setTimeout(
    () => setCounter(counter + 1),
    1000
  )

  return (
    <div>{counter}</div>
  )
}
```
**Every time the setCounter modifies the state** it causes the component to **re-render**. The value of the state will be incremented again after one second, and this will continue to repeat for as long as the application is running.
