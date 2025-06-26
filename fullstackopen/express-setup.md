# Express.js Backend Setup Guide

Complete setup guide for a Full Stack Open Express.js backend with authentication, database integration, and all necessary configurations.

## Project Initialization

Create a new directory and initialize the project:

```bash
mkdir notes-backend
cd notes-backend
npm init -y
```

## Dependencies Installation

### Core Dependencies

```bash
npm install express mongoose cors dotenv bcrypt jsonwebtoken
```

### Development Dependencies

```bash
npm install --save-dev nodemon eslint @eslint/js jest supertest cross-env
```

### Dependencies Breakdown

**Production Dependencies:**
- `express` - Web framework
- `mongoose` - MongoDB object modeling
- `cors` - Cross-Origin Resource Sharing
- `dotenv` - Environment variables
- `bcrypt` - Password hashing
- `jsonwebtoken` - JWT token implementation

**Development Dependencies:**
- `nodemon` - Auto-restart server on file changes
- `eslint` & `@eslint/js` - Code linting
- `jest` - Testing framework
- `supertest` - HTTP testing
- `cross-env` - Cross-platform environment variables

## Package.json Configuration

Update your `package.json` with the following scripts and configuration:

```json
{
  "name": "notes-backend",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "NODE_ENV=production node index.js",
    "dev": "NODE_ENV=development nodemon index.js",
    "test": "NODE_ENV=test jest --verbose --runInBand",
    "lint": "eslint ."
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.5.0",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1",
    "bcrypt": "^5.1.1",
    "jsonwebtoken": "^9.0.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1",
    "eslint": "^8.50.0",
    "@eslint/js": "^8.50.0",
    "jest": "^29.7.0",
    "supertest": "^6.3.3",
    "cross-env": "^7.0.3"
  },
  "jest": {
    "testEnvironment": "node",
    "globalTeardown": "./tests/teardown.js"
  }
}
```

## Project Structure

Create the following directory structure:

```
notes-backend/
├── controllers/
│   ├── notes.js
│   ├── users.js
│   └── login.js
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
├── requests.rest
├── .env
├── eslint.config.js
├── .gitignore
├── app.js
├── index.js
└── package.json
```

## Environment Configuration

Create `.env` file:

```
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/noteApp?retryWrites=true&w=majority
TEST_MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/testNoteApp?retryWrites=true&w=majority
PORT=3001
SECRET=your-secret-key-for-jwt-tokens
```

Create `.gitignore` file:

```
node_modules
.env
*.log
dist
build
```

## Configuration Files

### utils/config.js

```javascript
require('dotenv').config()

const PORT = process.env.PORT || 3001

const MONGODB_URI = process.env.NODE_ENV === 'test' 
  ? process.env.TEST_MONGODB_URI
  : process.env.MONGODB_URI

module.exports = {
  MONGODB_URI,
  PORT
}
```

### utils/logger.js

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

### utils/middleware.js

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
    return response.status(400).json({ error: 'token missing or invalid' })
  } else if (error.name === 'TokenExpiredError') {
    return response.status(401).json({ error: 'token expired' })
  }

  next(error)
}

const tokenExtractor = (request, response, next) => {
  const authorization = request.get('authorization')
  if (authorization && authorization.startsWith('Bearer ')) {
    request.token = authorization.replace('Bearer ', '')
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
## Database Models

### models/note.js

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

### models/user.js

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
    delete returnedObject.passwordHash
  }
})

const User = mongoose.model('User', userSchema)

module.exports = User
```

## Controllers

### controllers/notes.js

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

### controllers/users.js

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

### controllers/login.js

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

## Main Application Files

### app.js

```javascript
require('dotenv').config()
const config = require('./utils/config')
const express = require('express')
const app = express()
const cors = require('cors')
const notesRouter = require('./controllers/notes')
const usersRouter = require('./controllers/users')
const loginRouter = require('./controllers/login')
const middleware = require('./utils/middleware')
const logger = require('./utils/logger')
const mongoose = require('mongoose')

mongoose.set('strictQuery', false)

logger.info('connecting to', config.MONGODB_URI)

mongoose.connect(config.MONGODB_URI)
  .then(() => {
    logger.info('connected to MongoDB')
  })
  .catch((error) => {
    logger.error('error connecting to MongoDB:', error.message)
  })

app.use(cors())
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

### index.js

```javascript
const app = require('./app')
const config = require('./utils/config')
const logger = require('./utils/logger')

app.listen(config.PORT, () => {
  logger.info(`Server running on port ${config.PORT}`)
})
```

## ESLint Configuration

Create `eslint.config.js`:

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

## Testing Setup

### Basic Test Example - tests/note_api.test.js

```javascript
const mongoose = require('mongoose')
const supertest = require('supertest')
const app = require('../app')
const api = supertest(app)
const Note = require('../models/note')
const User = require('../models/user')

const initialNotes = [
  {
    content: 'HTML is easy',
    important: false,
  },
  {
    content: 'Browser can execute only JavaScript',
    important: true,
  },
]

beforeEach(async () => {
  await Note.deleteMany({})
  await User.deleteMany({})

  const user = new User({ username: 'root', passwordHash: 'sekret' })
  await user.save()

  for (let note of initialNotes) {
    let noteObject = new Note(note)
    noteObject.user = user._id
    await noteObject.save()
  }
})

test('notes are returned as json', async () => {
  await api
    .get('/api/notes')
    .expect(200)
    .expect('Content-Type', /application\/json/)
})

test('all notes are returned', async () => {
  const response = await api.get('/api/notes')
  expect(response.body).toHaveLength(initialNotes.length)
})

afterAll(async () => {
  await mongoose.connection.close()
})
```

### Test Teardown - tests/teardown.js

```javascript
module.exports = async () => {
  console.log('Test teardown completed')
}
```

## API Testing with REST Client

Create `requests.rest` file:

```http
### Get all notes
GET http://localhost:3001/api/notes

### Get a specific note
GET http://localhost:3001/api/notes/{{noteId}}

### Create a new user
POST http://localhost:3001/api/users
Content-Type: application/json

{
  "username": "testuser",
  "name": "Test User",
  "password": "testpassword"
}

### Login
POST http://localhost:3001/api/login
Content-Type: application/json

{
  "username": "testuser",
  "password": "testpassword"
}

### Create a new note (requires authentication)
POST http://localhost:3001/api/notes
Content-Type: application/json
Authorization: Bearer {{token}}

{
  "content": "This is a test note",
  "important": true
}

### Update a note
PUT http://localhost:3001/api/notes/{{noteId}}
Content-Type: application/json

{
  "content": "Updated content",
  "important": false
}

### Delete a note
DELETE http://localhost:3001/api/notes/{{noteId}}

### Get all users
GET http://localhost:3001/api/users
```

## Running the Application

### Development Mode
```bash
npm run dev
```

### Production Mode
```bash
npm start
```

### Running Tests
```bash
npm test
```

### ESLint Check
```bash
npm run lint
```

## Key Features Included

✅ **Express.js Server** with proper structure  
✅ **MongoDB Integration** with Mongoose  
✅ **User Authentication** with bcrypt and JWT  
✅ **CORS Configuration** for frontend integration  
✅ **Environment Variables** with dotenv  
✅ **Error Handling** middleware  
✅ **Request Logging** middleware  
✅ **Token Extraction** middleware  
✅ **ESLint Configuration** for code quality  
✅ **Testing Setup** with Jest and Supertest  
✅ **REST API** endpoints for CRUD operations  
✅ **Password Hashing** with bcrypt  
✅ **JWT Token Authentication**  
✅ **Database Population** (populate references)

This setup provides a complete Full Stack Open backend with all necessary configurations and dependencies for building a production-ready Express.js application.
