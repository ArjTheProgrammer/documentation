# PART 1: Introduction to React

**NOTE**: Full set up of React project is still tentative for 
part 5 to part 7 (testing react apps, advance state management,
React Router, custom hooks, styling apps with css, and webpack is 
still not finished)

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
> **OR** you can make a **SEPARATE FILE** like this:

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

