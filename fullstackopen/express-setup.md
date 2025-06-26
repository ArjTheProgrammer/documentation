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

Install Express.js and MongoDB dependencies:

```bash
npm install express mongoose
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
npm install --save-dev eslint @eslint/js
```

## Development Dependencies

For development, Node.js has built-in watch functionality that automatically restarts the server when files change. No additional dependencies needed for this feature.

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
    "dev": "node --watch index.js",
    "lint": "eslint .",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.5.0"
  },
  "devDependencies": {
    "eslint": "^8.50.0",
    "@eslint/js": "^8.50.0"
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

Create the main application file `index.js`:

```javascript
const express = require('express')
const mongoose = require('mongoose')
const config = require('./utils/config')
const logger = require('./utils/logger')
const middleware = require('./utils/middleware')
const notesRouter = require('./controllers/notes')

const app = express()

logger.info('connecting to', config.MONGODB_URI)

mongoose
  .connect(config.MONGODB_URI)
  .then(() => {
    logger.info('connected to MongoDB')
  })
  .catch((error) => {
    logger.error('error connection to MongoDB:', error.message)
  })

app.use(express.static('dist'))
app.use(express.json())
app.use(middleware.requestLogger)

app.use('/api/notes', notesRouter)

app.use(middleware.unknownEndpoint)
app.use(middleware.errorHandler)

module.exports = app
```

Create the notes controller file `controllers/notes.js`:

```javascript
const notesRouter = require('express').Router()
const Note = require('../models/note')

notesRouter.get('/', (request, response) => {
  Note.find({}).then(notes => {
    response.json(notes)
  })
})

notesRouter.get('/:id', (request, response, next) => {
  Note.findById(request.params.id)
    .then(note => {
      if (note) {
        response.json(note)
      } else {
        response.status(404).end()
      }
    })
    .catch(error => next(error))
})

notesRouter.post('/', (request, response, next) => {
  const body = request.body

  const note = new Note({
    content: body.content,
    important: body.important || false,
  })

  note.save()
    .then(savedNote => {
      response.json(savedNote)
    })
    .catch(error => next(error))
})

notesRouter.delete('/:id', (request, response, next) => {
  Note.findByIdAndDelete(request.params.id)
    .then(() => {
      response.status(204).end()
    })
    .catch(error => next(error))
})

notesRouter.put('/:id', (request, response, next) => {
  const { content, important } = request.body

  Note.findById(request.params.id)
    .then(note => {
      if (!note) {
        return response.status(404).end()
      }

      note.content = content
      note.important = important

      return note.save().then((updatedNote) => {
        response.json(updatedNote)
      })
    })
    .catch(error => next(error))
})

module.exports = notesRouter
```

## Express Routes Explained

Express routes define how your application responds to client requests to specific endpoints. Each route consists of:

- **Path**: The URL endpoint (e.g., `/api/notes`)
- **HTTP Method**: GET, POST, PUT, DELETE, etc.
- **Handler Function**: The function that executes when the route is matched

### HTTP Methods and Their Purpose

- **GET**: Retrieve data (should not modify server state)
- **POST**: Create new resources
- **PUT**: Update existing resources (replace entire resource)
- **PATCH**: Partial updates to existing resources
- **DELETE**: Remove resources

### Route Parameters

Access dynamic parts of the URL using route parameters:

```javascript
// Route parameter example
app.get('/api/notes/:id', (request, response) => {
  const id = request.params.id  // Access the :id parameter
  // ... handle request
})

// Multiple parameters
app.get('/api/users/:userId/notes/:noteId', (request, response) => {
  const { userId, noteId } = request.params
  // ... handle request
})
```

### Query Parameters

Access query string parameters from the URL:

```javascript
// URL: /api/notes?important=true&limit=10
app.get('/api/notes', (request, response) => {
  const { important, limit } = request.query
  
  let result = notes
  
  if (important !== undefined) {
    result = result.filter(note => note.important === (important === 'true'))
  }
  
  if (limit) {
    result = result.slice(0, Number(limit))
  }
  
  response.json(result)
})
```

### Request Body Handling

Process JSON data sent in request bodies:

```javascript
// Ensure you have JSON middleware
app.use(express.json())

app.post('/api/notes', (request, response) => {
  const { content, important } = request.body
  
  // Validation
  if (!content || content.trim() === '') {
    return response.status(400).json({
      error: 'Content cannot be empty'
    })
  }
  
  // Process the data...
})
```

### Error Handling in Routes

Implement proper error handling:

```javascript
app.get('/api/notes/:id', (request, response) => {
  try {
    const id = Number(request.params.id)
    
    if (isNaN(id)) {
      return response.status(400).json({ error: 'Invalid ID format' })
    }
    
    const note = notes.find(note => note.id === id)
    
    if (!note) {
      return response.status(404).json({ error: 'Note not found' })
    }
    
    response.json(note)
  } catch (error) {
    response.status(500).json({ error: 'Something went wrong' })
  }
})
```

### Organizing Routes with Express Router

For larger applications, organize routes using Express Router:

```javascript
// routes/notes.js
const notesRouter = require('express').Router()

notesRouter.get('/', (request, response) => {
  response.json(notes)
})

notesRouter.get('/:id', (request, response) => {
  // ... handle GET by ID
})

notesRouter.post('/', (request, response) => {
  // ... handle POST
})

notesRouter.put('/:id', (request, response) => {
  // ... handle PUT
})

notesRouter.delete('/:id', (request, response) => {
  // ... handle DELETE
})

module.exports = notesRouter

// In your main app.js
const notesRouter = require('./routes/notes')
app.use('/api/notes', notesRouter)
```

## Running the Application

### Development Mode
Start the development server (with automatic restart on file changes using Node.js built-in watch mode):

```bash
npm run dev
```

### Production Mode
Start the server in production mode:

```bash
npm start
```

## Testing the API

You can test your API endpoints using tools like curl, Postman, or VS Code REST Client:

### GET Requests
- `GET http://localhost:3001/` - Returns "Hello World!"
- `GET http://localhost:3001/api/notes` - Returns all notes
- `GET http://localhost:3001/api/notes/1` - Returns note with ID 1
- `GET http://localhost:3001/api/notes?important=true` - Returns only important notes

### POST Request (Create Note)
```bash
curl -X POST http://localhost:3001/api/notes \
  -H "Content-Type: application/json" \
  -d '{"content": "This is a new note", "important": true}'
```

### PUT Request (Update Note)
```bash
curl -X PUT http://localhost:3001/api/notes/1 \
  -H "Content-Type: application/json" \
  -d '{"content": "Updated note content", "important": false}'
```

### DELETE Request (Remove Note)
```bash
curl -X DELETE http://localhost:3001/api/notes/1
```

### Using VS Code REST Client

Create a `requests.rest` file for testing:

```http
### Get all notes
GET http://localhost:3001/api/notes

### Get a specific note
GET http://localhost:3001/api/notes/1

### Register a new user
POST http://localhost:3001/api/users
Content-Type: application/json

{
  "username": "testuser",
  "name": "Test User",
  "password": "testpassword123"
}

### Login
POST http://localhost:3001/api/login
Content-Type: application/json

{
  "username": "testuser",
  "password": "testpassword123"
}

### Create a new note (requires authentication)
POST http://localhost:3001/api/notes
Content-Type: application/json
Authorization: Bearer YOUR_JWT_TOKEN_HERE

{
  "content": "This is a test note requiring authentication",
  "important": true
}

### Update a note
PUT http://localhost:3001/api/notes/1
Content-Type: application/json

{
  "content": "Updated content",
  "important": false
}

### Delete a note
DELETE http://localhost:3001/api/notes/1

### Get all users
GET http://localhost:3001/api/users
```

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

Create a users controller `controllers/users.js` for user registration with bcrypt:

```javascript
const bcrypt = require('bcrypt')
const usersRouter = require('express').Router()
const User = require('../models/user')

usersRouter.get('/', async (request, response) => {
  const users = await User
    .find({}).populate('notes', { content: 1, important: 1 })

  response.json(users)
})

usersRouter.post('/', async (request, response, next) => {
  const { username, name, password } = request.body

  if (!password || password.length < 3) {
    return response.status(400).json({
      error: 'password must be at least 3 characters long'
    })
  }

  const saltRounds = 10
  const passwordHash = await bcrypt.hash(password, saltRounds)

  const user = new User({
    username,
    name,
    passwordHash,
  })

  try {
    const savedUser = await user.save()
    response.status(201).json(savedUser)
  } catch (error) {
    next(error)
  }
})

module.exports = usersRouter
```

Create a login controller `controllers/login.js` for authentication:

```javascript
const jwt = require('jsonwebtoken')
const bcrypt = require('bcrypt')
const loginRouter = require('express').Router()
const User = require('../models/user')

loginRouter.post('/', async (request, response) => {
  const { username, password } = request.body

  const user = await User.findOne({ username })
  const passwordCorrect = user === null
    ? false
    : await bcrypt.compare(password, user.passwordHash)

  if (!(user && passwordCorrect)) {
    return response.status(401).json({
      error: 'invalid username or password'
    })
  }

  const userForToken = {
    username: user.username,
    id: user._id,
  }

  const token = jwt.sign(userForToken, process.env.SECRET, { expiresIn: '1h' })

  response
    .status(200)
    .send({ token, username: user.username, name: user.name })
})

module.exports = loginRouter
```

### JWT Authentication

Create middleware for token authentication `utils/middleware.js`:

```javascript
const logger = require('./logger')
const jwt = require('jsonwebtoken')
const User = require('../models/user')

const requestLogger = (request, response, next) => {
  logger.info('Method:', request.method)
  logger.info('Path:  ', request.path)
  logger.info('Body:  ', request.body)
  logger.info('---')
  next()
}

const unknownEndpoint = (request, response) => {
  response.status(404).send({ error: 'unknown endpoint' })
}

const errorHandler = (error, request, response, next) => {
  logger.error(error.message)

  if (error.name === 'CastError') {
    return response.status(400).send({ error: 'malformatted id' })
  } else if (error.name === 'ValidationError') {
    return response.status(400).json({ error: error.message })
  } else if (error.name === 'JsonWebTokenError') {
    return response.status(401).json({ error: 'invalid token' })
  } else if (error.name === 'TokenExpiredError') {
    return response.status(401).json({ error: 'token expired' })
  }

  next(error)
}

const tokenExtractor = (request, response, next) => {
  const authorization = request.get('authorization')
  if (authorization && authorization.toLowerCase().startsWith('bearer ')) {
    request.token = authorization.substring(7)
  }
  next()
}

const userExtractor = async (request, response, next) => {
  if (request.token) {
    const decodedToken = jwt.verify(request.token, process.env.SECRET)
    if (decodedToken.id) {
      request.user = await User.findById(decodedToken.id)
    }
  }
  next()
}

module.exports = {
  requestLogger,
  unknownEndpoint,
  errorHandler,
  tokenExtractor,
  userExtractor
}
```

Update the notes controller to use authentication `controllers/notes.js`:

```javascript
const notesRouter = require('express').Router()
const Note = require('../models/note')
const User = require('../models/user')
const jwt = require('jsonwebtoken')

notesRouter.get('/', async (request, response) => {
  const notes = await Note
    .find({}).populate('user', { username: 1, name: 1 })

  response.json(notes)
})

notesRouter.get('/:id', async (request, response, next) => {
  try {
    const note = await Note.findById(request.params.id)
    if (note) {
      response.json(note)
    } else {
      response.status(404).end()
    }
  } catch (error) {
    next(error)
  }
})

notesRouter.post('/', async (request, response, next) => {
  const body = request.body

  if (!request.token) {
    return response.status(401).json({ error: 'token missing' })
  }

  try {
    const decodedToken = jwt.verify(request.token, process.env.SECRET)
    if (!decodedToken.id) {
      return response.status(401).json({ error: 'token invalid' })
    }

    const user = await User.findById(decodedToken.id)

    const note = new Note({
      content: body.content,
      important: body.important || false,
      user: user._id
    })

    const savedNote = await note.save()
    user.notes = user.notes.concat(savedNote._id)
    await user.save()

    response.status(201).json(savedNote)
  } catch (error) {
    next(error)
  }
})

notesRouter.delete('/:id', async (request, response, next) => {
  try {
    await Note.findByIdAndDelete(request.params.id)
    response.status(204).end()
  } catch (error) {
    next(error)
  }
})

notesRouter.put('/:id', async (request, response, next) => {
  const { content, important } = request.body

  try {
    const updatedNote = await Note.findByIdAndUpdate(
      request.params.id,
      { content, important },
      { new: true, runValidators: true, context: 'query' }
    )

    if (updatedNote) {
      response.json(updatedNote)
    } else {
      response.status(404).end()
    }
  } catch (error) {
    next(error)
  }
})

module.exports = notesRouter
```

Update your main `index.js` to include the new routes and middleware:

```javascript
const express = require('express')
const mongoose = require('mongoose')
const config = require('./utils/config')
const logger = require('./utils/logger')
const middleware = require('./utils/middleware')
const notesRouter = require('./controllers/notes')
const usersRouter = require('./controllers/users')
const loginRouter = require('./controllers/login')

const app = express()

logger.info('connecting to', config.MONGODB_URI)

mongoose
  .connect(config.MONGODB_URI)
  .then(() => {
    logger.info('connected to MongoDB')
  })
  .catch((error) => {
    logger.error('error connection to MongoDB:', error.message)
  })

app.use(express.static('dist'))
app.use(express.json())
app.use(middleware.requestLogger)
app.use(middleware.tokenExtractor)

app.use('/api/notes', middleware.userExtractor, notesRouter)
app.use('/api/users', usersRouter)
app.use('/api/login', loginRouter)

app.use(middleware.unknownEndpoint)
app.use(middleware.errorHandler)

module.exports = app
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

## Required Model Files

Create the User model `models/user.js`:

```javascript
const mongoose = require('mongoose')

const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    unique: true,
    minlength: 3
  },
  name: String,
  passwordHash: String,
  notes: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Note'
    }
  ],
})

userSchema.set('toJSON', {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString()
    delete returnedObject._id
    delete returnedObject.__v
    // the passwordHash should not be revealed
    delete returnedObject.passwordHash
  }
})

const User = mongoose.model('User', userSchema)

module.exports = User
```

Update the Note model `models/note.js` to include user reference:

```javascript
const mongoose = require('mongoose')

const noteSchema = new mongoose.Schema({
  content: {
    type: String,
    required: true,
    minlength: 5
  },
  important: Boolean,
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
})

noteSchema.set('toJSON', {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString()
    delete returnedObject._id
    delete returnedObject.__v
  }
})

module.exports = mongoose.model('Note', noteSchema)
```

Create configuration file `utils/config.js`:

```javascript
require('dotenv').config()

const PORT = process.env.PORT
const MONGODB_URI = process.env.NODE_ENV === 'test'
  ? process.env.TEST_MONGODB_URI
  : process.env.MONGODB_URI

module.exports = {
  MONGODB_URI,
  PORT
}
```

Create logger utility `utils/logger.js`:

```javascript
const info = (...params) => {
  if (process.env.NODE_ENV !== 'test') {
    console.log(...params)
  }
}

const error = (...params) => {
  if (process.env.NODE_ENV !== 'test') {
    console.error(...params)
  }
}

module.exports = {
  info, error
}
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
