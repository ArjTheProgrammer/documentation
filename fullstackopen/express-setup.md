# Express.js Setup Guide

This guide covers setting up a basic Express.js server, following the Full Stack Open course methodology.

## Initial Project Setup

Create a new directory for your backend application and initialize it:

```bash
mkdir notes-backend
cd notes-backend
npm init -y
```

The `-y` flag accepts all default values for the npm initialization.

## Installing Dependencies

Install Express.js as a dependency:

```bash
npm install express
```

### Authentication Dependencies

For user authentication, install bcrypt for password hashing and jsonwebtoken for JWT tokens:

```bash
npm install bcrypt jsonwebtoken
```

### Additional Useful Dependencies

Install other commonly used packages:

```bash
npm install cors dotenv
npm install --save-dev nodemon eslint @eslint/js
```

## Development Dependencies

For development, install nodemon to automatically restart the server when files change:

```bash
npm install --save-dev nodemon
```

### ESLint Setup

Install ESLint for code linting and formatting:

```bash
npm install --save-dev eslint @eslint/js
```

Create `.eslintrc.js` configuration file:

```javascript
const js = require('@eslint/js')

module.exports = [
  js.configs.recommended,
  {
    files: ['**/*.js'],
    languageOptions: {
      sourceType: 'commonjs',
      globals: {
        process: 'readonly'
      },
      ecmaVersion: 'latest'
    },
    rules: {
      'indent': ['error', 2],
      'linebreak-style': ['error', 'unix'],
      'quotes': ['error', 'single'],
      'semi': ['error', 'never'],
      'eqeqeq': 'error',
      'no-trailing-spaces': 'error',
      'object-curly-spacing': ['error', 'always'],
      'arrow-spacing': ['error', { 'before': true, 'after': true }],
      'no-console': 0
    }
  },
  {
    ignores: ['dist/**', 'build/**']
  }
]
```

## Package.json Scripts

Update your `package.json` scripts section:

```json
{
  "name": "notes-backend",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "lint": "eslint .",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

## Project File Structure

Organize your project with a clear file structure:

```
notes-backend/
├── controllers/
│   ├── notes.js
│   └── users.js
├── models/
│   ├── note.js
│   └── user.js
├── utils/
│   ├── config.js
│   ├── logger.js
│   └── middleware.js
├── tests/
│   ├── note_api.test.js
│   └── user_api.test.js
├── .env
├── .eslintrc.js
├── .gitignore
├── index.js
└── package.json
```

### File Structure Explanation

- **controllers/**: Route handlers and business logic
- **models/**: Database schemas and models
- **utils/**: Utility functions, configuration, and middleware
- **tests/**: Test files
- **index.js**: Main application entry point

## Basic Express Server

Create an `index.js` file with a basic Express server:

```javascript
const express = require('express')
const app = express()

// Middleware to parse JSON
app.use(express.json())

// Sample data
let notes = [
  {
    id: 1,
    content: "HTML is easy",
    important: false
  },
  {
    id: 2,
    content: "Browser can execute only JavaScript",
    important: true
  },
  {
    id: 3,
    content: "GET and POST are the most important methods of HTTP protocol",
    important: true
  }
]

// Routes
app.get('/', (request, response) => {
  response.send('<h1>Hello World!</h1>')
})

app.get('/api/notes', (request, response) => {
  response.json(notes)
})

app.get('/api/notes/:id', (request, response) => {
  const id = Number(request.params.id)
  const note = notes.find(note => note.id === id)
  
  if (note) {
    response.json(note)
  } else {
    response.status(404).end()
  }
})

// Start server
const PORT = process.env.PORT || 3001
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`)
})
```

## Running the Application

### Development Mode
Start the development server (with automatic restart on file changes):

```bash
npm run dev
```

### Production Mode
Start the server in production mode:

```bash
npm start
```

## Testing the API

You can test your API endpoints:

- `GET http://localhost:3001/` - Returns "Hello World!"
- `GET http://localhost:3001/api/notes` - Returns all notes
- `GET http://localhost:3001/api/notes/1` - Returns note with ID 1

## Additional Configuration

### Environment Variables

Create a `.env` file for environment-specific configuration:

```bash
npm install dotenv
```

Add to your `index.js`:

```javascript
require('dotenv').config()
```

Example `.env` file:

```
NODE_ENV=development
PORT=3001
MONGODB_URI=mongodb://localhost:27017/notes
TEST_MONGODB_URI=mongodb://localhost:27017/notes-test
SECRET=your-secret-key-for-jwt
```

### Password Hashing with bcrypt

Example of using bcrypt for password hashing:

```javascript
const bcrypt = require('bcrypt')

// Hash password before saving user
const saltRounds = 10
const passwordHash = await bcrypt.hash(password, saltRounds)

// Compare password during login
const passwordCorrect = user === null
  ? false
  : await bcrypt.compare(password, user.passwordHash)
```

### JWT Authentication

Example of using jsonwebtoken for authentication:

```javascript
const jwt = require('jsonwebtoken')

// Generate token
const userForToken = {
  username: user.username,
  id: user._id,
}

const token = jwt.sign(userForToken, process.env.SECRET)

// Verify token (middleware)
const getTokenFrom = request => {
  const authorization = request.get('authorization')
  if (authorization && authorization.toLowerCase().startsWith('bearer ')) {
    return authorization.substring(7)
  }
  return null
}

const token = getTokenFrom(request)
const decodedToken = jwt.verify(token, process.env.SECRET)
```

### CORS (Cross-Origin Resource Sharing)

If you plan to connect a frontend, install and configure CORS:

```bash
npm install cors
```

Add to your `index.js`:

```javascript
const cors = require('cors')
app.use(cors())
```

## Git Setup

Initialize git repository and create `.gitignore`:

```bash
git init
```

Create `.gitignore` file:

```
node_modules
.env
*.log
dist
build
```

### Running ESLint

Check your code for linting errors:

```bash
npm run lint
```

## Next Steps

- Add POST, PUT, and DELETE routes for full CRUD functionality
- Implement middleware for logging requests
- Add error handling middleware
- Connect to a database (MongoDB with Mongoose)
- Add input validation
- Implement user authentication with bcrypt and JWT
- Add unit and integration tests
- Set up CI/CD pipeline
