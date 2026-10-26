# ACS 4330 Advanced - Week 3: Authentication and Authorization

**Estimated time:** 5 hours for the lesson + 2–3 hours on your final project

**Prerequisites:** Completed Weeks 1 and 2. You have a MongoDB-backed Pets server and a React + Apollo Client frontend.

> **Already did "Authentication with GraphQL" (Lesson 12) in ACS 4330?** Skim Part 1, but switch your errors to `GraphQLError` as shown here. Part 2 (authorization) is new.

<!-- > -->

## Learning Outcomes

By the end of this lesson, you should be able to:

1. Describe how JWT authentication works in a GraphQL API
2. Write `register` and `login` mutations that return a token
3. Verify a JWT in the Apollo Server `context` function
4. Protect resolvers with reusable `requireAuth` and `requireRole` helpers
5. Enforce ownership: users can only change records they created
6. Send the token from Apollo Client and handle login/logout in React

<!-- > -->

## Review

From memory:

- What does the `context` argument in a resolver contain in your Pets server?
- When does the `context` function run — once at startup, or once per request?
- After a mutation, what are the ways to update the Apollo cache?

<!-- > -->

## Authentication vs. Authorization

- **Authentication** — *Who are you?* Proving identity: logging in.
- **Authorization** — *What are you allowed to do?* Checking permissions: "only admins can add shelters," "only the person who listed a pet can delete it."

You need authentication first. Authorization builds on it.

<!-- > -->

# Part 1 — Authentication

<!-- > -->

## 🔐 How Auth Works in GraphQL

1. A user registers or logs in with a **mutation**
2. The server returns a **JWT** (JSON Web Token) — a signed string that says "this is user X"
3. The client stores the token and sends it in the `Authorization` header of every request
4. The server reads the header in the **`context` function**, verifies the token, and puts the user on context
5. Resolvers check `context.user` before doing anything sensitive

You don't need any special GraphQL middleware. Context does it all.

<!-- > -->

## Install Dependencies

In `pets/server`:

```bash
npm install jsonwebtoken@9 bcryptjs@3
```

- `jsonwebtoken` — creates and verifies JWTs
- `bcryptjs` — hashes passwords. **Never store plain-text passwords.** (`bcryptjs` is a pure-JavaScript version of `bcrypt`, so there's nothing to compile.)

Add a secret to `.env`:

```
JWT_SECRET=a-long-random-string-change-this
```

Generate a good one with `node -e "console.log(crypto.randomUUID() + crypto.randomUUID())"`.

<!-- > -->

## Step 1 — Add a User Model

`models/User.js`:

```js
import mongoose from 'mongoose'

const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true, lowercase: true, trim: true },
  passwordHash: { type: String, required: true },
  role: { type: String, enum: ['user', 'admin'], default: 'user' },
})

export default mongoose.model('User', userSchema)
```

`passwordHash` is in the database schema but will **not** be in the GraphQL schema. Clients can never ask for it.

<!-- > -->

## Step 2 — Update the GraphQL Schema

Add to `schema.js`:

```graphql
type User {
  id: ID!
  email: String!
  role: String!
}

type AuthPayload {
  token: String!
  user: User!
}

type Query {
  me: User
  # ...existing queries
}

type Mutation {
  register(email: String!, password: String!): AuthPayload!
  login(email: String!, password: String!): AuthPayload!
  # ...existing mutations
}
```

`me` returns the logged-in user, or `null` if there's no valid token.

<!-- > -->

## Step 3 — Auth Helpers

Create `auth.js`. It holds token signing and the checks you'll reuse in every protected resolver:

```js
import jwt from 'jsonwebtoken'
import { GraphQLError } from 'graphql'

export function signToken(user) {
  return jwt.sign(
    { userId: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: '7d' }
  )
}

export function getUserFromToken(authHeader = '') {
  const token = authHeader.replace('Bearer ', '')
  if (!token) return null
  try {
    return jwt.verify(token, process.env.JWT_SECRET)  // { userId, role, iat, exp }
  } catch {
    return null  // expired or tampered token: treat as logged out
  }
}

export function requireAuth(context) {
  if (!context.user) {
    throw new GraphQLError('You must be logged in', {
      extensions: { code: 'UNAUTHENTICATED' },
    })
  }
  return context.user
}
```

`GraphQLError` lets you attach a `code` to the error. The client can check the code instead of matching message text.

<!-- > -->

## Step 4 — Verify the Token in Context

Update the `context` function in `server.js`:

```js
import User from './models/User.js'
import { getUserFromToken } from './auth.js'

// ...

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 },
  context: async ({ req }) => ({
    user: getUserFromToken(req.headers.authorization),
    Pet,
    Shelter,
    User,
  }),
})
```

The context function runs on **every request**. With no token or a bad token, `context.user` is `null`. That's intentional: public queries like `pets` should still work for visitors.

<!-- > -->

## Step 5 — Register, Login, and `me`

In `resolvers.js`:

```js
import bcrypt from 'bcryptjs'
import { GraphQLError } from 'graphql'
import { signToken } from './auth.js'

const resolvers = {
  Query: {
    me: (_, __, { user, User }) => (user ? User.findById(user.userId) : null),
    // ...existing queries
  },
  Mutation: {
    register: async (_, { email, password }, { User }) => {
      if (password.length < 8) {
        throw new GraphQLError('Password must be at least 8 characters', {
          extensions: { code: 'BAD_USER_INPUT' },
        })
      }
      const existing = await User.findOne({ email })
      if (existing) {
        throw new GraphQLError('Email already in use', {
          extensions: { code: 'BAD_USER_INPUT' },
        })
      }
      const passwordHash = await bcrypt.hash(password, 10)
      const user = await User.create({ email, passwordHash })
      return { token: signToken(user), user }
    },

    login: async (_, { email, password }, { User }) => {
      const user = await User.findOne({ email })
      const valid = user && (await bcrypt.compare(password, user.passwordHash))
      if (!valid) {
        throw new GraphQLError('Invalid email or password', {
          extensions: { code: 'UNAUTHENTICATED' },
        })
      }
      return { token: signToken(user), user }
    },
    // ...existing mutations
  },
  // ...Pet and Shelter resolvers
}
```

`login` gives the **same** error for a wrong email and a wrong password. A different message for each would tell an attacker which emails have accounts.

<!-- > -->

## Step 6 — Protect a Mutation

Only logged-in users can add pets:

```js
import { requireAuth, signToken } from './auth.js'

// ...
addPet: (_, args, context) => {
  requireAuth(context)
  return context.Pet.create(args)
},
```

One line per protected resolver, and the error is the same everywhere.

<!-- > -->

## Test in Apollo Sandbox

**1. Register:**

```graphql
mutation {
  register(email: "test@example.com", password: "secret123") {
    token
    user { id email role }
  }
}
```

**2. Copy the token.** In Sandbox, open the **Headers** panel and add:

```
Authorization: Bearer YOUR_TOKEN_HERE
```

**3. Run `me`** — it returns your user. **Run `addPet`** — it succeeds.

**4. Remove the header** and run `addPet` again. You get `"You must be logged in"` with `code: "UNAUTHENTICATED"`.

Paste your token into https://jwt.io. The payload (`userId`, `role`) is **readable by anyone**. The signature only stops people from *changing* it. Never put secrets in a JWT.

<!-- > -->

# Part 2 — Authorization

<!-- > -->

## Roles: Admin-Only Actions

Add a role check to `auth.js`:

```js
export function requireRole(context, role) {
  const user = requireAuth(context)
  if (user.role !== role) {
    throw new GraphQLError('You do not have permission to do that', {
      extensions: { code: 'FORBIDDEN' },
    })
  }
  return user
}
```

Only admins can add shelters:

```js
addShelter: (_, args, context) => {
  requireRole(context, 'admin')
  return context.Shelter.create(args)
},
```

**UNAUTHENTICATED** means "log in first." **FORBIDDEN** means "you're logged in, but you can't do this." Clients handle the two differently: one shows a login form, the other shows a message.

<!-- > -->

## Making an Admin

`register` always creates a `user`. Never let a client choose its own role. To make an admin:

1. Open your cluster in Atlas → **Browse Collections** → `users`
2. Edit your user document and change `role` to `"admin"`
3. **Log in again** to get a new token

Why log in again? The role is stored **inside the token**. Your old token still says `user` until it expires. That's the trade-off of putting the role in the JWT: it's fast (no database lookup on every request), but role changes only apply after the next login. If you need role changes to apply right away, look up the user in the context function instead.

<!-- > -->

## Ownership: Only the Creator Can Change It

Record who created each pet. In `models/Pet.js`:

```js
createdBy: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
```

Set it in `addPet` from the token. **Never** take it from the client's arguments:

```js
addPet: (_, args, context) => {
  const user = requireAuth(context)
  return context.Pet.create({ ...args, createdBy: user.userId })
},
```

<!-- > -->

Then check ownership before changing or deleting. Add a helper to `auth.js`:

```js
export function requireOwnerOrAdmin(context, doc) {
  const user = requireAuth(context)
  const isOwner = doc.createdBy?.toString() === user.userId
  if (!isOwner && user.role !== 'admin') {
    throw new GraphQLError('You can only change your own records', {
      extensions: { code: 'FORBIDDEN' },
    })
  }
  return user
}
```

`createdBy` is an ObjectId and `userId` is a string, so convert with `.toString()` before comparing. The `?.` handles pets created before you added `createdBy`. Only admins can change those.

<!-- > -->

Use it in `updatePet` and `deletePet`:

```js
updatePet: async (_, { id, ...fields }, context) => {
  const pet = await context.Pet.findById(id)
  if (!pet) return null
  requireOwnerOrAdmin(context, pet)
  pet.set(fields)
  return pet.save()
},

deletePet: async (_, { id }, context) => {
  const pet = await context.Pet.findById(id)
  if (!pet) return false
  requireOwnerOrAdmin(context, pet)
  await pet.deleteOne()
  return true
},
```

The pattern is: **load → check → act**. You have to load the record before you can check who owns it.

<!-- > -->

## Exposing the Creator

Let clients see who listed each pet. In `schema.js`:

```graphql
type Pet {
  # ...existing fields
  createdBy: User
}
```

In `resolvers.js`, add to the `Pet` resolvers:

```js
createdBy: (pet, _, { User }) => (pet.createdBy ? User.findById(pet.createdBy) : null),
```

Now anyone can see every user's email, which may be more than you want. Two common fixes are hiding `email` from other users with a field resolver, or exposing a separate public type (e.g. `PublicUser { id displayName }`). Challenge 4 covers this.

<!-- > -->

# Part 3 — The React Client

<!-- > -->

## Send the Token with Every Request

Update `src/apolloClient.js`:

```js
import { ApolloClient, HttpLink, InMemoryCache } from '@apollo/client'
import { SetContextLink } from '@apollo/client/link/context'

const httpLink = new HttpLink({ uri: 'http://localhost:4000/' })

const authLink = new SetContextLink(({ headers }) => {
  const token = localStorage.getItem('token')
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
    },
  }
})

const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache(),
})

export default client
```

Links run in order. `authLink` adds the header, then `httpLink` sends the request. The callback runs before **every** request, so it always reads the current token.

<!-- > -->

## Queries for Auth

Add to `src/queries.js`:

```js
export const ME = gql`
  query Me {
    me { id email role }
  }
`

export const LOGIN = gql`
  mutation Login($email: String!, $password: String!) {
    login(email: $email, password: $password) {
      token
      user { id email role }
    }
  }
`
```

Write `REGISTER` the same way.

<!-- > -->

## Log In and Log Out

`src/Auth.jsx`:

```jsx
import { useState } from 'react'
import { useApolloClient, useMutation, useQuery } from '@apollo/client/react'
import { LOGIN, ME } from './queries.js'

export default function Auth() {
  const client = useApolloClient()
  const { data } = useQuery(ME)
  const [login, { error }] = useMutation(LOGIN)
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')

  const handleLogin = async (e) => {
    e.preventDefault()
    try {
      const { data } = await login({ variables: { email, password } })
      localStorage.setItem('token', data.login.token)
      await client.resetStore()  // refetch everything as the new user
    } catch {
      // error is shown below
    }
  }

  const handleLogout = async () => {
    localStorage.removeItem('token')
    await client.resetStore()  // clear cached data from the old user
  }

  if (data?.me) {
    return (
      <p>
        Logged in as {data.me.email} ({data.me.role})
        <button onClick={handleLogout}>Log out</button>
      </p>
    )
  }

  return (
    <form onSubmit={handleLogin}>
      <input type="email" value={email} onChange={e => setEmail(e.target.value)} placeholder="Email" required />
      <input type="password" value={password} onChange={e => setPassword(e.target.value)} placeholder="Password" required />
      <button>Log in</button>
      {error && <p role="alert">{error.message}</p>}
    </form>
  )
}
```

Render `<Auth />` at the top of `App`.

<!-- > -->

## Why `resetStore()`?

The cache holds data fetched as the **previous** user, such as `me`. `client.resetStore()` clears the cache and re-runs every active query with the new token. Without it, the UI shows the old user's data until you refresh the page.

> **About `localStorage`:** Any JavaScript on your page can read `localStorage`, so an XSS bug could leak the token. Production apps often use `httpOnly` cookies instead. `localStorage` is fine for this course. Just know the trade-off.

<!-- > -->

## Challenges

<!-- > -->

**Challenge 1 — Register, Login, `me`**

1. Add the `User` model, `auth.js`, schema types, and resolvers
2. Test `register`, `login`, and `me` in Sandbox, with and without the `Authorization` header
3. Check the `users` collection in Atlas: the password is stored as a hash

<!-- > -->

**Challenge 2 — Authorization**

1. Require login for `addPet`; set `createdBy` from the token
2. Make `addShelter` admin-only
3. Apply `requireOwnerOrAdmin` to `updatePet` and `deletePet`
4. Test with **two** users in Sandbox: user A can't delete user B's pet, and an admin can delete any pet

<!-- > -->

**Challenge 3 — React Login**

1. Add `SetContextLink` to the client
2. Add the `Auth` component with login, logout, and the current user
3. Add a register form (or a toggle between register and login)
4. Only show Delete buttons on pets the current user created. Add `createdBy { id }` to `GET_PETS` and compare it with `me.id`
5. In the Network tab, confirm the `authorization` header is sent

<!-- > -->

**Challenge 4 — Stretch: Field-Level Privacy**

Make `User.email` visible only to that user and to admins. Everyone else gets `null`. Hint: add a `User: { email: (user, _, context) => ... }` resolver and make `email` nullable in the schema.

<!-- > -->

## Final Project Milestone

1. Add `register`, `login`, and `me`
2. Protect every mutation that changes data
3. Add at least **one** authorization rule beyond "logged in": ownership or a role
4. Add login/logout to your client, and hide actions the user can't perform

<!-- > -->

## After This Lesson

- Push `pets/server/` and `pets/client/`
- Push your updated `final-project/`
- Mark Week 3 complete in the course tracker

<!-- > -->

### Evaluate Your Work

| | Does not meet expectations | Meets expectations | Exceeds expectations |
|:---:|:---:|:---:|:---:|
| Auth mutations | `register`/`login` missing or don't return a token | Both work; passwords hashed; token is a valid JWT | Duplicate email and weak password rejected; login error doesn't reveal which part was wrong |
| Context | Token not read in context | `context.user` set from a valid token | Missing or invalid tokens give `null` without throwing |
| Authorization | No resolvers protected | Mutations require login via a shared helper | Ownership or role rule enforced; correct `UNAUTHENTICATED` vs `FORBIDDEN` codes |
| React client | Token not sent | Auth link sends token; login stores it | Logout clears token and resets store; UI hides actions the user can't take |

<!-- > -->

## Resources

- https://www.apollographql.com/docs/apollo-server/security/authentication
- https://www.apollographql.com/docs/apollo-server/data/errors
- https://www.apollographql.com/docs/react/networking/authentication
- https://jwt.io — decode and inspect a token
- https://www.npmjs.com/package/jsonwebtoken
- https://www.npmjs.com/package/bcryptjs
