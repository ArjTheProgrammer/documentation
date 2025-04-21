
# PART 1: Introduction to React

> **NOTE:** At this time, I am studying redux and currently on part 6 of fullstackopen.

---

## Starting a New React Project

You can quickly create a React project using Vite:

```bash
npm create vite@latest introdemo -- --template react
```

---

## Simplifying `main.jsx`

Here’s a minimal version of the `main.jsx` file:

```jsx
import ReactDOM from 'react-dom/client'
import App from './App'

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

---

### `App.jsx` Now Looks Like This

```jsx
const App = () => {
  return (
    <div>
      <p>Hello world</p>
    </div>
  )
}

export default App
```

---

## Creating Multiple React Components

You can define multiple components in the same file and use them repeatedly:

```jsx
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

### Or Create a Separate Component File

**Hello.jsx**
```jsx
const Hello = () => {
  return (
    <div>
      <p>Hello world</p>
    </div>
  )
}

export default Hello
```

**App.jsx**
```jsx
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

---

## Props: Passing Data to Components

**Props** are used to pass data into components:

```jsx
const Hello = (props) => {
  console.log(props)
  return (
    <div>
      <p>Hello {props.name}, you are {props.age} years old</p>
    </div>
  )
}

const App = () => {
  const name = 'Peter'
  const age = 10

  return (
    <div>
      <h1>Greetings</h1>
      <Hello name="Maya" age={26 + 10} />
      <Hello name={name} age={age} />
    </div>
  )
}
```

- The `Hello` component receives `props.name` and `props.age`.
- Values passed as props can be strings, variables, or expressions (wrapped in curly braces).

---

### Destructuring Props

You can simplify props handling by destructuring:

```jsx
const Hello = ({ name, age }) => {
  return (
    <div>
      <p>Hello {name}, you are {age} years old</p>
    </div>
  )
}
```

> **Note:** Do not try to render **objects** directly as children in JSX — React will throw an error.

---

## Stateful Components

The **state** of a component is used to store dynamic data that can change over time. When state changes, React re-renders the component.

```jsx
import { useState } from 'react'

const App = () => {
  const [counter, setCounter] = useState(0)

  setTimeout(() => setCounter(counter + 1), 1000)

  return (
    <div>{counter}</div>
  )
}

export default App
```

### Explanation:

- `useState(0)` initializes the state variable `counter` to 0.
- `setCounter` is the function used to update the state.
- Each time `setCounter` is called, the component re-renders.

### Infinite Re-Render Warning

Don't do this:

```jsx
<button onClick={setCounter(counter + 1)}> 
  plus
</button>
```

This immediately calls `setCounter` during rendering, causing an infinite loop of updates.

### Correct Way: Use an Arrow Function

```jsx
<button onClick={() => setCounter(counter + 1)}>
  plus
</button>
```

Now `setCounter` only runs when the button is clicked.

---

## [Rules of Hooks](https://react.dev/warnings/invalid-hook-call-warning#breaking-rules-of-hooks)

Hooks must always be called at the top level of a React function component. **Never call hooks inside loops, conditions, or nested functions.**

Invalid usage examples:

```jsx
const App = () => {
  const [age, setAge] = useState(0)
  const [name, setName] = useState('Juha Tauriainen')

  if (age > 10) {
    // ❌ Invalid
    const [foobar, setFoobar] = useState(null)
  }

  for (let i = 0; i < age; i++) {
    // ❌ Invalid
    const [rightWay, setRightWay] = useState(false)
  }

  const notGood = () => {
    // ❌ Invalid
    const [x, setX] = useState(-1000)
  }

  return (
    // ...
  )
}
```

Always call hooks **unconditionally** and **directly inside the component function**.