# ACS 4330 - GraphQL + MongoDB

**Estimated time:** 4–5 hours

**Prerequisites:** Completed Lesson 9. You have a working Apollo Server with in-memory data and a project proposal.

<!-- > -->

## Learning Outcomes

By the end of this lesson, you should be able to:

1. Connect Apollo Server to MongoDB using Mongoose
2. Replace in-memory arrays with Mongoose models
3. Write async resolvers that read and write to a database
4. Handle the `_id` → `id` conversion between MongoDB and GraphQL

<!-- > -->

## Review

Write these from memory:

- How do you define a resolver that accepts arguments?
- What does `async`/`await` do in a resolver?
- What is the `context` argument used for?

<!-- > -->

## 🗄️ Why Persist Data?

In-memory arrays reset every time your server restarts. For a real app — and your final project — you need data that survives restarts.

MongoDB is a document database that stores JSON-like objects. Mongoose is a Node.js library that adds schemas and models on top of MongoDB.

**Two schemas, different purposes:**

| | GraphQL Schema | Mongoose Schema |
|---|---|---|
| What it defines | The shape of your API | The shape of your database documents |
| Where it lives | `typeDefs` string | `new mongoose.Schema({...})` |
| Who uses it | Apollo Server | MongoDB via Mongoose |

They often look similar, but they're separate things.

<!-- > -->

## Setup

<!-- > -->

### 1. Get a MongoDB database

Use [MongoDB Atlas](https://www.mongodb.com/atlas) (free tier):

1. Create an account
2. Create a free cluster
3. Click **Connect** → **Drivers** → copy the connection string
4. Replace `<password>` with your database user password

Or install [MongoDB Community Server](https://www.mongodb.com/try/download/community) locally — connection string is `mongodb://127.0.0.1:27017/mydb`.

<!-- > -->

### 2. Install Mongoose

```bash
npm install mongoose
```

<!-- > -->

### 3. Add connection string to `.env`

```
MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/mydb
```

Add `.env` to `.gitignore` — never commit credentials.

<!-- > -->

### 4. Connect in `server.js`

```js
import { ApolloServer } from '@apollo/server'
import { startStandaloneServer } from '@apollo/server/standalone'
import mongoose from 'mongoose'
import 'dotenv/config'

await mongoose.connect(process.env.MONGO_URI)
console.log('Connected to MongoDB')

// typeDefs, resolvers, server setup below...
```

Connect **before** starting the server. If `mongoose.connect` throws, your server shouldn't start.

<!-- > -->

## Mongoose Models

A Mongoose model is a class that maps to a MongoDB collection. Define a schema, then create the model.

```js
import mongoose from 'mongoose'

const petSchema = new mongoose.Schema({
  name: { type: String, required: true },
  species: { type: String, required: true },
})

const Pet = mongoose.model('Pet', petSchema)

export default Pet
```

Put each model in its own file under `models/` — e.g. `models/Pet.js`.

<!-- > -->

## Replacing In-Memory Resolvers

### Before (in-memory)

```js
let petList = [
  { id: '1', name: 'Biscuit', species: 'cat' },
]

const resolvers = {
  Query: {
    pets: () => petList,
    pet: (_, { id }) => petList.find(p => p.id === id),
  },
  Mutation: {
    addPet: (_, { name, species }) => {
      const pet = { id: String(petList.length + 1), name, species }
      petList.push(pet)
      return pet
    },
  }
}
```

<!-- > -->

### After (MongoDB)

```js
import Pet from './models/Pet.js'

const resolvers = {
  Query: {
    pets: async () => Pet.find(),
    pet: async (_, { id }) => Pet.findById(id),
  },
  Mutation: {
    addPet: async (_, { name, species }) => {
      return Pet.create({ name, species })
    },
    updatePet: async (_, { id, name, species }) => {
      return Pet.findByIdAndUpdate(
        id,
        { ...(name && { name }), ...(species && { species }) },
        { new: true }  // return the updated document
      )
    },
    deletePet: async (_, { id }) => {
      const result = await Pet.findByIdAndDelete(id)
      return result !== null
    }
  }
}
```

All resolvers are now `async` — Mongoose operations return promises.

<!-- > -->

## The `_id` → `id` Problem

MongoDB stores documents with `_id`, not `id`. Your GraphQL schema uses `id: ID!`.

Mongoose solves this automatically. Every Mongoose document has a virtual `id` getter that returns `_id.toString()`. Apollo's default resolvers read `obj.id`, which resolves to the Mongoose virtual.

**You don't need to do anything extra** — it works out of the box. But know why: if you ever bypass Mongoose and use the raw MongoDB driver, you'll need to convert manually.

<!-- > -->

## Full Example

`models/Pet.js`:

```js
import mongoose from 'mongoose'

const petSchema = new mongoose.Schema({
  name: { type: String, required: true },
  species: { type: String, required: true },
})

export default mongoose.model('Pet', petSchema)
```

`server.js`:

```js
import { ApolloServer } from '@apollo/server'
import { startStandaloneServer } from '@apollo/server/standalone'
import mongoose from 'mongoose'
import 'dotenv/config'
import Pet from './models/Pet.js'

await mongoose.connect(process.env.MONGO_URI)
console.log('Connected to MongoDB')

const typeDefs = `#graphql
  type Pet {
    id: ID!
    name: String!
    species: String!
  }

  type Query {
    pets: [Pet!]!
    pet(id: ID!): Pet
  }

  type Mutation {
    addPet(name: String!, species: String!): Pet!
    updatePet(id: ID!, name: String, species: String): Pet
    deletePet(id: ID!): Boolean!
  }
`

const resolvers = {
  Query: {
    pets: async () => Pet.find(),
    pet: async (_, { id }) => Pet.findById(id),
  },
  Mutation: {
    addPet: async (_, { name, species }) => Pet.create({ name, species }),
    updatePet: async (_, { id, name, species }) => {
      return Pet.findByIdAndUpdate(
        id,
        { ...(name && { name }), ...(species && { species }) },
        { new: true }
      )
    },
    deletePet: async (_, { id }) => {
      const result = await Pet.findByIdAndDelete(id)
      return result !== null
    }
  }
}

const server = new ApolloServer({ typeDefs, resolvers })

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 }
})

console.log(`Server ready at: ${url}`)
```

<!-- > -->

## Passing Models via Context

When your app has multiple models, pass them through context instead of importing them into every resolver file:

```js
import Pet from './models/Pet.js'
import User from './models/User.js'

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 },
  context: async () => ({ Pet, User })
})
```

Resolver:

```js
pets: async (_, __, context) => context.Pet.find(),
```

This makes resolvers easier to test and keeps imports centralized.

<!-- > -->

## Challenges

<!-- > -->

**Challenge 1 — Set up MongoDB**

1. Create a MongoDB Atlas cluster (or start a local MongoDB instance)
2. Create a `.env` with `MONGO_URI`
3. Add `mongoose` to your server
4. Connect before starting — verify "Connected to MongoDB" logs on startup

<!-- > -->

**Challenge 2 — Migrate the Pet Server**

Starting from the pet server you built in Lesson 6:

1. Create `models/Pet.js` with a Mongoose schema
2. Replace `petList` array with Mongoose queries in all resolvers
3. Test every query and mutation in Apollo Sandbox:
   - `pets` — list all
   - `pet(id)` — get one
   - `addPet` — create
   - `updatePet` — update
   - `deletePet` — delete
4. Restart the server and verify data persists

<!-- > -->

**Challenge 3 — Add a Second Model**

Add an `Owner` type:

```graphql
type Owner {
  id: ID!
  name: String!
  pets: [Pet!]!
}
```

1. Create `models/Owner.js`
2. Add `ownerId` to your `Pet` Mongoose schema
3. Write a nested resolver: `Pet.owner` and `Owner.pets`
4. Add `addOwner` and `assignPet(petId, ownerId)` mutations

<!-- > -->

**Challenge 4 — Migrate Your Final Project**

Apply what you've learned to your final project server:

1. Create a `models/` folder with a Mongoose model for each type
2. Replace all in-memory arrays with Mongoose queries
3. Pass models through context
4. Test all queries and mutations — data should persist across server restarts

<!-- > -->

## After This Lesson

- Submit your MongoDB-backed server to GradeScope
- Your final project server should now persist data

<!-- > -->

### Evaluate Your Work

| | Does not meet expectations | Meets expectations | Exceeds expectations |
|:---:|:---:|:---:|:---:|
| MongoDB connection | Server doesn't connect to MongoDB | Server connects on startup; error is thrown if connection fails | Connection string in `.env`; models passed via context |
| Mongoose models | No models defined | At least one model with correct schema | Multiple models; required fields marked; relationships via IDs |
| Resolvers | Resolvers still use in-memory arrays | All CRUD resolvers use Mongoose; async/await used correctly | Handles null results; uses `{ new: true }` on updates |

<!-- > -->

## Resources

- https://mongoosejs.com/docs/guide.html
- https://www.mongodb.com/atlas
- https://www.apollographql.com/docs/apollo-server/data/fetching-data
