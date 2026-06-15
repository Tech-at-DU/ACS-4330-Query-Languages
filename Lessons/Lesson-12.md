# ACS 4330 - Authentication with GraphQL

**Estimated time:** 4–5 hours

**Prerequisites:** Completed Lessons 9 and 10. You have a MongoDB-backed Apollo Server and a React + Apollo Client frontend.

<!-- > -->

## Learning Outcomes

By the end of this lesson, you should be able to:

1. Describe how JWT authentication works in a GraphQL API
2. Write `register` and `login` mutations that return a token
3. Read and verify a JWT token in the Apollo Server `context` function
4. Send an auth token from Apollo Client using an auth link
5. Protect resolvers so only authenticated users can run them

<!-- > -->

## Review

From memory:

- What does the `context` argument in a resolver contain?
- How do you define the `context` function in `startStandaloneServer`?
- What is `async`/`await` doing in a resolver?

<!-- > -->

## 🔐 How Auth Works in GraphQL

GraphQL auth follows the same pattern as REST — a token proves who you are — but the implementation is different:

1. User registers or logs in via a **mutation**
2. Server returns a **JWT token**
3. Client stores the token and sends it in every request header
4. Server reads the header in the **`context` function**, verifies the token, and attaches the user to context
5. Resolvers check `context.user` before doing sensitive operations

No special GraphQL middleware needed — just context.

<!-- > -->

## Install Dependencies

```bash
npm install jsonwebtoken bcrypt
```

- `jsonwebtoken` — creates and verifies JWT tokens
- `bcrypt` — hashes passwords (never store plain text)

Add a secret to `.env`:

```
JWT_SECRET=some-long-random-string-change-this
```

<!-- > -->

## Step 1 — Add a User Model

`models/User.js`:

```js
import mongoose from 'mongoose'

const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  passwordHash: { type: String, required: true },
})

export default mongoose.model('User', userSchema)
```

Never store the plain-text password. Store only the hash.

<!-- > -->

## Step 2 — Update the Schema

Add `User`, `AuthPayload`, and auth mutations:

```graphql
type User {
  id: ID!
  email: String!
}

type AuthPayload {
  token: String!
  user: User!
}

type Query {
  me: User         # returns the currently authenticated user
  # ... your existing queries
}

type Mutation {
  register(email: String!, password: String!): AuthPayload!
  login(email: String!, password: String!): AuthPayload!
  # ... your existing mutations
}
```

<!-- > -->

## Step 3 — Write the Auth Resolvers

```js
import jwt from 'jsonwebtoken'
import bcrypt from 'bcrypt'
import User from './models/User.js'

const resolvers = {
  Query: {
    me: async (_, __, context) => {
      if (!context.user) return null
      return User.findById(context.user.userId)
    },
    // ... existing queries
  },
  Mutation: {
    register: async (_, { email, password }) => {
      const existing = await User.findOne({ email })
      if (existing) throw new Error('Email already in use')

      const passwordHash = await bcrypt.hash(password, 10)
      const user = await User.create({ email, passwordHash })

      const token = jwt.sign(
        { userId: user.id },
        process.env.JWT_SECRET,
        { expiresIn: '7d' }
      )

      return { token, user }
    },

    login: async (_, { email, password }) => {
      const user = await User.findOne({ email })
      if (!user) throw new Error('No account with that email')

      const valid = await bcrypt.compare(password, user.passwordHash)
      if (!valid) throw new Error('Incorrect password')

      const token = jwt.sign(
        { userId: user.id },
        process.env.JWT_SECRET,
        { expiresIn: '7d' }
      )

      return { token, user }
    },

    // ... existing mutations
  }
}
```

<!-- > -->

## Step 4 — Verify the Token in Context

Update your `context` function in `server.js`:

```js
import jwt from 'jsonwebtoken'

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 },
  context: async ({ req }) => {
    const auth = req.headers.authorization || ''
    const token = auth.replace('Bearer ', '')

    try {
      const user = jwt.verify(token, process.env.JWT_SECRET)
      return { user }
    } catch {
      return { user: null }
    }
  }
})
```

If no token or an invalid token is sent, `context.user` is `null`. This is intentional — unauthenticated requests should still work for public queries.

<!-- > -->

## Step 5 — Protect a Mutation

Check `context.user` at the top of any mutation that requires authentication:

```js
addPet: async (_, { name, species }, context) => {
  if (!context.user) {
    throw new Error('You must be logged in to add a pet')
  }
  return Pet.create({ name, species })
},
```

Apollo Client will receive this as an error in the `error` object from `useMutation`. Handle it in your UI.

<!-- > -->

## Step 6 — Send the Token from Apollo Client

Apollo Client needs to attach the token to every request. Use an **auth link**:

```bash
# Already installed with @apollo/client — no extra package needed
```

Update `src/apolloClient.js`:

```js
import { ApolloClient, InMemoryCache, createHttpLink } from '@apollo/client'
import { setContext } from '@apollo/client/link/context'

const httpLink = createHttpLink({
  uri: 'http://localhost:4000/',
})

const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('token')
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
    }
  }
})

const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache(),
})

export default client
```

<!-- > -->

## Step 7 — Store and Clear the Token in React

After a successful `login` or `register` mutation, store the token:

```jsx
const [login] = useMutation(LOGIN)

const handleLogin = async (email, password) => {
  const { data } = await login({ variables: { email, password } })
  localStorage.setItem('token', data.login.token)
}
```

On logout, clear it:

```js
localStorage.removeItem('token')
```

<!-- > -->

## Test the Full Flow in Apollo Sandbox

**1. Register:**

```graphql
mutation {
  register(email: "test@example.com", password: "secret123") {
    token
    user {
      id
      email
    }
  }
}
```

**2. Copy the token. Add an Authorization header in Apollo Sandbox:**

```json
{
  "Authorization": "Bearer YOUR_TOKEN_HERE"
}
```

**3. Run a protected mutation — it should succeed.**

**4. Remove the header and try again — it should throw an error.**

<!-- > -->

## Challenge 1 — Implement Register and Login

1. Add the `User` model, schema types, and auth resolvers
2. Test `register` and `login` in Apollo Sandbox
3. Verify the token is returned and the user is created in MongoDB

<!-- > -->

## Challenge 2 — Protect a Mutation

1. Add an auth check to at least one mutation in your project
2. Test with a valid token — mutation succeeds
3. Test without a token — mutation throws an error

<!-- > -->

## Challenge 3 — Wire Up the React Client

1. Update `src/apolloClient.js` to use the auth link
2. Add a login form to your React app
3. Store the token in `localStorage` on successful login
4. Verify the token is sent with requests (check the Network tab — look at request headers)

<!-- > -->

## Challenge 4 — Add `me` Query to the UI

1. Add a `me` query to your React app
2. Display the logged-in user's email when a token is present
3. Show a login form when no token exists
4. Add a logout button that clears the token and reloads

<!-- > -->

## After This Lesson

- Submit your auth-enabled server and React client to GradeScope
- Your final project should have working login/register by this point

<!-- > -->

### Evaluate Your Work

| | Does not meet expectations | Meets expectations | Exceeds expectations |
|:---:|:---:|:---:|:---:|
| Auth mutations | `register`/`login` missing or don't return a token | Both mutations work; token is a valid JWT; password is hashed | Duplicate email handled; invalid password gives clear error |
| Context | Token not read in context | Token verified in context; `context.user` is set on valid requests | `context.user` is `null` on missing/invalid token without throwing |
| Protected mutations | No mutations protected | At least one mutation checks `context.user` | Error message is clear; unprotected queries still work without a token |
| React client | Token not sent from client | Auth link sends token in headers; token stored after login | Logout clears token; `me` query shows current user |

<!-- > -->

## Resources

- https://jwt.io — paste a token here to decode and inspect it
- https://www.npmjs.com/package/jsonwebtoken
- https://www.npmjs.com/package/bcrypt
- https://www.apollographql.com/docs/react/networking/authentication
