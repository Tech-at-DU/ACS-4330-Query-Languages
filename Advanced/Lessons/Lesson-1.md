# ACS 4330 Advanced - Week 1: MongoDB and Data Modeling

**Estimated time:** 4–5 hours for the lesson + 2–3 hours on your final project

**Prerequisites:** Completed ACS 4330 Query Languages. You can write an Apollo Server with queries, mutations, and nested resolvers.

> **Already did "GraphQL + MongoDB" (Lesson 10) in ACS 4330?** Skim Part 1 and reuse your server. Part 2 is new.

<!-- > -->

## Learning Outcomes

By the end of this lesson, you should be able to:

1. Connect Apollo Server to MongoDB using Mongoose
2. Replace in-memory arrays with Mongoose models
3. Write async resolvers that read and write to a database
4. Model a relationship between two collections and resolve it with nested resolvers
5. Use Mongoose validation to reject bad data before it reaches the database

<!-- > -->

## Review

Write these from memory before you start:

- How do you define a resolver that accepts arguments?
- What does `async`/`await` do in a resolver?
- What is the `context` argument used for?
- What is a nested resolver? When does it run?

<!-- > -->

## The Project for This Course

Every week you will do two things:

1. **Follow along** with a small **Pets** app (a server in Week 1, plus a React client from Week 2). The lessons give you all the code you need.
2. **Apply** what you learned to your **final project**. See [the final project spec](../Assignments/FinalProjectSpec.md).

Set up your course repo like this:

```
acs-4330-advanced/
  pets/
    server/
    client/       ← added in Week 2
  final-project/
    server/
    client/
```

<!-- > -->

# Part 1 — Connect GraphQL to MongoDB

<!-- > -->

## 🗄️ Why Persist Data?

In-memory arrays reset every time your server restarts. A real app needs data that survives restarts.

MongoDB is a document database that stores JSON-like objects. Mongoose is a Node.js library that adds schemas and models on top of MongoDB.

**Two schemas, different purposes:**

| | GraphQL Schema | Mongoose Schema |
|---|---|---|
| What it defines | The shape of your API | The shape of your database documents |
| Where it lives | `typeDefs` string | `new mongoose.Schema({...})` |
| Who uses it | Apollo Server | MongoDB via Mongoose |

They often look similar, but they're separate things. A field can exist in one and not the other — for example, a password hash lives in the database but never in the API.

<!-- > -->

## Setup

<!-- > -->

### 1. Get a MongoDB database

Use [MongoDB Atlas](https://www.mongodb.com/atlas) (free tier). You'll need a hosted database in Week 7 to deploy anyway.

1. Create an account
2. Create a free cluster
3. Under **Database Access**, create a database user and password
4. Under **Network Access**, add your current IP address
5. Click **Connect** → **Drivers** → copy the connection string
6. Replace `<password>` with your database user password, and add a database name after `.net/` (e.g. `.net/pets`)

Or install [MongoDB Community Server](https://www.mongodb.com/try/download/community) locally — the connection string is `mongodb://127.0.0.1:27017/pets`.

<!-- > -->

### 2. Create the server project

```bash
mkdir -p pets/server
cd pets/server
npm init -y
npm pkg set type=module
npm pkg set scripts.start="node server.js"
npm pkg set scripts.dev="node --watch server.js"
npm install @apollo/server graphql mongoose dotenv
```

`npm run dev` restarts the server whenever you save a file.

<!-- > -->

### 3. Add the connection string to `.env`

```
MONGO_URI=mongodb+srv://username:password@cluster0.xxxxx.mongodb.net/pets
```

Create a `.gitignore` that excludes `node_modules` and `.env`. **Never commit credentials.**

<!-- > -->

## File Structure

The server grows every week, so split it into files now:

```
pets/server/
  models/
    Pet.js
  schema.js        ← GraphQL typeDefs
  resolvers.js     ← resolver map
  server.js        ← connects to MongoDB and starts Apollo Server
  .env
  .gitignore
  package.json
```

<!-- > -->

## Mongoose Models

A Mongoose model is a class that maps to a MongoDB collection. Define a schema, then create the model.

`models/Pet.js`:

```js
import mongoose from 'mongoose'

const petSchema = new mongoose.Schema({
  name: { type: String, required: true },
  species: { type: String, required: true },
})

export default mongoose.model('Pet', petSchema)
```

The model name `'Pet'` becomes the collection `pets` in MongoDB.

<!-- > -->

## The GraphQL Schema

`schema.js`:

```js
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

export default typeDefs
```

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

`resolvers.js`:

```js
const resolvers = {
  Query: {
    pets: (_, __, { Pet }) => Pet.find(),
    pet: (_, { id }, { Pet }) => Pet.findById(id),
  },
  Mutation: {
    addPet: (_, { name, species }, { Pet }) => Pet.create({ name, species }),

    updatePet: (_, { id, ...fields }, { Pet }) => {
      return Pet.findByIdAndUpdate(id, fields, {
        returnDocument: 'after',  // return the updated document, not the old one
        runValidators: true,      // apply schema validation to updates too
      })
    },

    deletePet: async (_, { id }, { Pet }) => {
      const result = await Pet.findByIdAndDelete(id)
      return result !== null
    },
  },
}

export default resolvers
```

Notes:

- Mongoose methods return promises. Apollo waits for a returned promise, so you only need `async`/`await` when you use the result inside the resolver, as `deletePet` does.
- `{ id, ...fields }` works because GraphQL only includes arguments the client actually sent. If the client sends only `name`, then `fields` is `{ name }`.
- The models come from the third argument, **context**. The next section shows why.

<!-- > -->

## Passing Models via Context

Pass your models through context instead of importing them into every resolver file:

`server.js`:

```js
import { ApolloServer } from '@apollo/server'
import { startStandaloneServer } from '@apollo/server/standalone'
import mongoose from 'mongoose'
import 'dotenv/config'
import typeDefs from './schema.js'
import resolvers from './resolvers.js'
import Pet from './models/Pet.js'

await mongoose.connect(process.env.MONGO_URI)
console.log('Connected to MongoDB')

const server = new ApolloServer({ typeDefs, resolvers })

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 },
  context: async () => ({ Pet }),
})

console.log(`Server ready at: ${url}`)
```

Connect **before** starting the server. If `mongoose.connect` throws, the server shouldn't start.

Context keeps imports in one place. It also makes resolvers easy to test, because a test can pass fake models in context.

<!-- > -->

## The `_id` → `id` Problem

MongoDB stores documents with `_id`, not `id`. Your GraphQL schema uses `id: ID!`.

Mongoose handles this for you. Every Mongoose document has a virtual `id` getter that returns `_id` as a string. Apollo's default resolver reads `pet.id`, which uses that virtual.

**You don't need to do anything extra.** But know why it works: if you ever use the raw MongoDB driver or `.lean()`, you get plain objects with no `id`, and you have to convert `_id` yourself.

<!-- > -->

## Try It

Run `npm run dev` and open http://localhost:4000 (Apollo Sandbox).

```graphql
mutation {
  addPet(name: "Biscuit", species: "cat") {
    id
    name
  }
}
```

Copy the `id` from the response. Then run `pets`, `pet(id: ...)`, `updatePet`, and `deletePet`. Stop and restart the server — your data is still there.

Look at the collection in Atlas (**Browse Collections**) to see the stored documents.

> **Gotcha:** `pet(id: "123")` throws a `CastError`, because `"123"` isn't a valid MongoDB ObjectId. Real IDs look like `"665f1c2e9b1e8a3d4c7f0a12"`.

<!-- > -->

# Part 2 — Data Modeling

<!-- > -->

## Embedding vs. Referencing

MongoDB gives you two ways to connect data:

| | Embed | Reference |
|---|---|---|
| How | Store the related object **inside** the document | Store the related document's **ID** |
| Example | A `Pet` with an array of `vaccinations` inside it | A `Pet` with a `shelterId` |
| Reads | One query gets everything | Needs a second query (or a nested resolver) |
| Good for | Data that belongs to one parent and is always read with it | Data shared by many documents or queried on its own |

**Rule of thumb:** If the child makes no sense without the parent, embed it. If the child is its own thing — something you list, search, or update on its own — reference it.

A shelter has many pets and is its own thing, so we'll **reference** it.

<!-- > -->

## Add a `Shelter` Model

`models/Shelter.js`:

```js
import mongoose from 'mongoose'

const shelterSchema = new mongoose.Schema({
  name: { type: String, required: true, trim: true },
  city: { type: String, required: true, trim: true },
})

export default mongoose.model('Shelter', shelterSchema)
```

<!-- > -->

## Reference the Shelter from `Pet`

Update `models/Pet.js` with a reference, plus some validation:

```js
import mongoose from 'mongoose'

const petSchema = new mongoose.Schema({
  name: { type: String, required: true, trim: true, maxlength: 50 },
  species: {
    type: String,
    required: true,
    enum: ['cat', 'dog', 'bird', 'rabbit', 'other'],
  },
  age: { type: Number, min: 0, max: 40 },
  shelterId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Shelter',
    index: true,
  },
})

export default mongoose.model('Pet', petSchema)
```

- `enum`, `min`, `max`, and `maxlength` are **validators**. `Pet.create({ species: 'dragon' })` throws, and Apollo returns the error message to the client.
- `ref: 'Shelter'` tells Mongoose which model the ID points to.
- `index: true` makes "find all pets in this shelter" fast. **Add an index to any field you query by often.**

<!-- > -->

## Update the GraphQL Schema

```graphql
type Shelter {
  id: ID!
  name: String!
  city: String!
  pets: [Pet!]!
}

type Pet {
  id: ID!
  name: String!
  species: String!
  age: Int
  shelter: Shelter
}

type Query {
  pets: [Pet!]!
  pet(id: ID!): Pet
  shelters: [Shelter!]!
  shelter(id: ID!): Shelter
}

type Mutation {
  addPet(name: String!, species: String!, age: Int, shelterId: ID): Pet!
  updatePet(id: ID!, name: String, species: String, age: Int, shelterId: ID): Pet
  deletePet(id: ID!): Boolean!
  addShelter(name: String!, city: String!): Shelter!
}
```

Notice that the database stores `shelterId` (an ID), but the API exposes `shelter` (an object). The nested resolver connects the two.

<!-- > -->

## Nested Resolvers

Add these to `resolvers.js`:

```js
const resolvers = {
  Query: {
    pets: (_, __, { Pet }) => Pet.find(),
    pet: (_, { id }, { Pet }) => Pet.findById(id),
    shelters: (_, __, { Shelter }) => Shelter.find(),
    shelter: (_, { id }, { Shelter }) => Shelter.findById(id),
  },
  Mutation: {
    addPet: (_, args, { Pet }) => Pet.create(args),
    // updatePet and deletePet stay the same
    addShelter: (_, args, { Shelter }) => Shelter.create(args),
  },
  Pet: {
    shelter: (pet, _, { Shelter }) => {
      if (!pet.shelterId) return null
      return Shelter.findById(pet.shelterId)
    },
  },
  Shelter: {
    pets: (shelter, _, { Pet }) => Pet.find({ shelterId: shelter.id }),
  },
}
```

Add `Shelter` to context in `server.js`:

```js
import Shelter from './models/Shelter.js'

// ...
  context: async () => ({ Pet, Shelter }),
```

<!-- > -->

## What About `populate()`?

Mongoose can also join referenced documents for you:

```js
Pet.find().populate('shelterId')  // shelterId becomes the full Shelter document
```

In a GraphQL server, **nested resolvers are usually the better choice:**

- A nested resolver only runs if the client asks for that field. `populate()` always runs.
- The API field name (`shelter`) and the database field (`shelterId`) stay separate.

Nested resolvers have one cost. Fetching 50 pets with their shelters runs 1 + 50 queries. That's the **N+1 problem**, and you'll fix it in Week 6.

<!-- > -->

## Try It

```graphql
mutation {
  addShelter(name: "Happy Tails", city: "Oakland") {
    id
  }
}
```

```graphql
mutation {
  addPet(name: "Biscuit", species: "cat", age: 3, shelterId: "PASTE_SHELTER_ID") {
    name
    shelter {
      name
      city
    }
  }
}
```

```graphql
query {
  shelters {
    name
    pets {
      name
      species
    }
  }
}
```

Now try `addPet(name: "Smaug", species: "dragon")`. Read the error message that comes back.

<!-- > -->

## Challenges

<!-- > -->

**Challenge 1 — Set up MongoDB**

1. Create a MongoDB Atlas cluster
2. Create `pets/server` with a `.env` containing `MONGO_URI`
3. Verify "Connected to MongoDB" logs on startup

<!-- > -->

**Challenge 2 — Pets CRUD**

1. Create `models/Pet.js`, `schema.js`, `resolvers.js`, and `server.js` as shown in Part 1
2. Test every operation in Apollo Sandbox: `pets`, `pet(id)`, `addPet`, `updatePet`, `deletePet`
3. Restart the server and verify the data persists

<!-- > -->

**Challenge 3 — Shelters**

1. Add the `Shelter` model and the validation shown in Part 2
2. Add the nested resolvers `Pet.shelter` and `Shelter.pets`
3. Create two shelters and at least five pets spread across them
4. Run the `shelters { name pets { name } }` query and verify the results
5. Trigger a validation error on purpose and note the message

<!-- > -->

**Challenge 4 — Stretch: Delete Safely**

What should happen to a shelter's pets when the shelter is deleted? Add a `deleteShelter(id: ID!): Boolean!` mutation and pick one behavior:

- Refuse to delete a shelter that still has pets (throw an error), **or**
- Unset `shelterId` on its pets (`Pet.updateMany({ shelterId: id }, { $unset: { shelterId: 1 } })`)

Write one sentence in your README explaining your choice.

<!-- > -->

## Final Project Milestone

Read the [final project spec](../Assignments/FinalProjectSpec.md). Then, for your project:

1. Write `final-project/PLAN.md` — your types, their fields, and the relationships between them. For each relationship, write whether you'll embed or reference it, and why.
2. Create a Mongoose model for each type, with validation for required fields
3. Replace any in-memory arrays with Mongoose queries, and pass models through context
4. Implement at least one relationship with nested resolvers

If you're reusing your ACS 4330 final project, this is mostly a migration. If you're starting fresh, keep the schema small — 2–3 types is plenty.

<!-- > -->

## After This Lesson

- Push `pets/server/` to your course repo
- Push `final-project/PLAN.md` and your updated `final-project/server/`
- Mark Week 1 complete in the course tracker
- Bring one question about your data model to the weekly check-in

<!-- > -->

### Evaluate Your Work

| | Does not meet expectations | Meets expectations | Exceeds expectations |
|:---:|:---:|:---:|:---:|
| MongoDB connection | Server doesn't connect to MongoDB | Connects on startup; connection string in `.env` | `.env` is gitignored; models passed via context |
| Resolvers | Resolvers still use in-memory arrays | All CRUD resolvers use Mongoose | Updates use `returnDocument: 'after'` and `runValidators`; missing documents return `null` |
| Relationships | No relationship between models | One relationship resolved with nested resolvers | Embed vs. reference choice explained in `PLAN.md`; referenced field is indexed |
| Validation | No validation | Required fields marked | Uses `enum`/`min`/`max` where they fit the data |

<!-- > -->

## Resources

- https://mongoosejs.com/docs/guide.html
- https://mongoosejs.com/docs/validation.html
- https://www.mongodb.com/docs/manual/data-modeling/
- https://www.mongodb.com/atlas
- https://www.apollographql.com/docs/apollo-server/data/fetching-data
