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

> **NOTE:** Do not render **OBJECTS** because it is not
as a react child