# ACS 4330 - Nested Resolvers

**Estimated time:** 3–4 hours

**Prerequisites:** Completed Lessons 2 and 6. Comfortable with Apollo Server schemas, resolvers, and mutations.

<!-- > -->

## Learning Objectives

1. Explain what a nested resolver is and why it's needed
2. Use the `parent` argument in a resolver
3. Write type-level resolvers that resolve relationship fields
4. Build a schema with connected types

<!-- > -->

## Before You Start

In Lesson 1 you ran this query against the Rick and Morty API:

```graphql
{
  character(id: 1) {
    name
    origin {
      name
    }
  }
}
```

A single query returned a character *and* their origin location — a nested type.

**Write down:** how do you think the server resolved the `origin` field? What had to happen between fetching the character and returning the location?

<!-- > -->

## What is a Nested Resolver?

<!-- > -->

So far every resolver you've written handles a top-level query:

```js
const resolvers = {
  Query: {
    character: (_, { id }) => characters.find(c => c.id === id),
  }
}
```

This works when all the data needed lives on the object you return. But what about a field that points to *another type*?

<!-- > -->

```graphql
type Character {
  id: ID!
  name: String!
  origin: Location   # <-- this is another type, not a scalar
}
```

The character object in your data might store a `locationId` number — not a full `Location` object. GraphQL needs to resolve that `origin` field separately.

<!-- > -->

That's what a **nested resolver** (also called a **field resolver**) does — it resolves a specific field on a type, receiving the parent object as its first argument.

```js
const resolvers = {
  Query: {
    character: (_, { id }) => characters.find(c => c.id === id)
  },
  Character: {                                    // <-- type name
    origin: (parent) => {                         // <-- field name
      return locations.find(l => l.id === parent.originId)
    }
  }
}
```

<!-- > -->

### The `parent` Argument

Every resolver receives `(parent, args, context, info)`.

At the `Query` level, `parent` is usually ignored (`_`). At the type level, **`parent` is the object returned by the parent resolver** — in this case, the character object.

```js
Character: {
  origin: (parent) => {
    // parent = { id: '1', name: 'Rick Sanchez', originId: '1', locationId: '3' }
    return locations.find(l => l.id === parent.originId)
  }
}
```

<!-- > -->

### When does GraphQL call a field resolver?

Only when the client asks for that field. If the query is:

```graphql
{ character(id: 1) { name } }
```

The `origin` resolver is **never called** — GraphQL only resolves what was requested. This is one of the key efficiency wins of GraphQL.

<!-- > -->

## Building It

<!-- > -->

### Setup

Create a new Apollo Server project (same setup as Lesson 2):

```bash
mkdir nested-resolvers
cd nested-resolvers
npm init -y
```

Add to `package.json`:
```json
{
  "type": "module",
  "scripts": {
    "start": "node --watch server.js"
  }
}
```

Install:
```bash
npm install @apollo/server
```

Create `server.js`.

<!-- > -->

### The Data

You'll build a small subset of the Rick and Morty schema with in-memory data. Add this to `server.js`:

```js
const characters = [
  { id: '1', name: 'Rick Sanchez',  status: 'Alive', originId: '1', locationId: '3' },
  { id: '2', name: 'Morty Smith',   status: 'Alive', originId: '2', locationId: '3' },
  { id: '3', name: 'Summer Smith',  status: 'Alive', originId: '20', locationId: '20' },
  { id: '4', name: 'Beth Smith',    status: 'Alive', originId: '20', locationId: '20' },
  { id: '5', name: 'Jerry Smith',   status: 'Alive', originId: '20', locationId: '20' },
]

const locations = [
  { id: '1',  name: 'Earth (C-137)',                dimension: 'Dimension C-137' },
  { id: '2',  name: 'Abadango',                     dimension: 'unknown' },
  { id: '3',  name: 'Citadel of Ricks',             dimension: 'unknown' },
  { id: '20', name: 'Earth (Replacement Dimension)', dimension: 'Replacement Dimension' },
]
```

Notice: characters store `originId` and `locationId` — just IDs, not full objects. The nested resolvers will look up the actual Location.

<!-- > -->

### The Schema

```js
const typeDefs = `#graphql
  type Character {
    id: ID!
    name: String!
    status: String!
    origin: Location
    location: Location
  }

  type Location {
    id: ID!
    name: String!
    dimension: String
    residents: [Character!]!
  }

  type Query {
    character(id: ID!): Character
    characters: [Character!]!
    location(id: ID!): Location
    locations: [Location!]!
  }
`
```

<!-- > -->

### Query Resolvers

Start with the top-level Query resolvers:

```js
const resolvers = {
  Query: {
    character: (_, { id }) => characters.find(c => c.id === id),
    characters: () => characters,
    location: (_, { id }) => locations.find(l => l.id === id),
    locations: () => locations,
  }
}
```

At this point, querying `character(id: "1") { name status }` works — but `origin { name }` would return `null` because the Character object only has `originId`, not a full Location.

<!-- > -->

### Character Field Resolvers

Add a `Character` block to your resolvers:

```js
const resolvers = {
  Query: {
    // ... existing queries
  },
  Character: {
    origin: (parent) => {
      return locations.find(l => l.id === parent.originId) || null
    },
    location: (parent) => {
      return locations.find(l => l.id === parent.locationId) || null
    }
  }
}
```

`parent` here is the character object returned by the Query resolver. The field resolver looks up the matching Location by ID.

<!-- > -->

### Location Field Resolvers

A Location can also resolve back to its residents (the characters currently there):

```js
const resolvers = {
  Query: { /* ... */ },
  Character: { /* ... */ },
  Location: {
    residents: (parent) => {
      return characters.filter(c => c.locationId === parent.id)
    }
  }
}
```

<!-- > -->

### Start the Server

```js
import { ApolloServer } from '@apollo/server'
import { startStandaloneServer } from '@apollo/server/standalone'

const server = new ApolloServer({ typeDefs, resolvers })

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 }
})

console.log(`Server ready at: ${url}`)
```

<!-- > -->

### Test Your Work

Open Apollo Sandbox at `http://localhost:4000` and try these queries:

```graphql
# Should resolve Rick's origin location
{
  character(id: "1") {
    name
    origin {
      name
      dimension
    }
  }
}
```

```graphql
# Should resolve all residents of the Citadel of Ricks
{
  location(id: "3") {
    name
    residents {
      name
      status
    }
  }
}
```

```graphql
# Multi-level nesting: location → residents → their origin
{
  location(id: "3") {
    name
    residents {
      name
      origin {
        name
      }
    }
  }
}
```

<!-- > -->

## The N+1 Problem

Brief heads-up for when you work with databases: if you query all characters and each needs to resolve its location, GraphQL calls the `location` resolver **once per character**. With 100 characters that's 100 database queries — the N+1 problem.

For now with in-memory data this doesn't matter. When you connect to a database (Lesson 10), tools like **DataLoader** solve this by batching multiple lookups into one. Just something to be aware of.

<!-- > -->

## Challenges

<!-- > -->

**Challenge 1 — Verify the Setup**

Get the following queries working in Apollo Sandbox:

1. `character(id: "1") { name origin { name } }`
2. `character(id: "2") { name location { name dimension } }`
3. `location(id: "3") { name residents { name } }`

<!-- > -->

**Challenge 2 — Add Episodes**

Add an `Episode` type and connect it to characters.

```graphql
type Episode {
  id: ID!
  name: String!
  episode: String!    # e.g. "S01E01"
  characters: [Character!]!
}
```

Add episode data (you can use real R&M episode names or make them up):

```js
const episodes = [
  { id: '1', name: 'Pilot', episode: 'S01E01', characterIds: ['1', '2', '3', '4', '5'] },
  { id: '2', name: 'Lawnmower Dog', episode: 'S01E02', characterIds: ['1', '2'] },
]
```

Write the resolver: `Episode.characters` should look up each character by id from `characterIds`.

<!-- > -->

**Challenge 3 — Characters Know Their Episodes**

Add an `episodes` field to `Character`:

```graphql
type Character {
  # ... existing fields
  episodes: [Episode!]!
}
```

Write the resolver: `Character.episodes` should return all episodes that include this character's id in `characterIds`.

<!-- > -->

**Challenge 4 — Query Deep Nesting**

Write a query in Apollo Sandbox that:

1. Gets all characters
2. For each character, gets their current location
3. For each location, gets the other residents

This exercises 3 levels of nesting in a single query.

<!-- > -->

**Challenge 5 — Add a Character**

Add a mutation `addCharacter` that creates a new character. The new character should accept `name`, `status`, `originId`, and `locationId`.

After adding a character, verify that:
- `location(id: <originId>) { residents { name } }` shows the new character

<!-- > -->

**Challenge 6 — Null Safety**

What happens if you query a character with a `locationId` that doesn't exist in your locations array?

Test it: change one character's `locationId` to `'999'` and query their location.

Handle this gracefully in your resolver so it returns `null` instead of throwing.

<!-- > -->

## After This Lesson

Submit your completed nested resolvers project to GradeScope.

Before moving on, you should be able to:
- [ ] Explain what the `parent` argument is and when it's used
- [ ] Write a resolver under a type name (e.g. `Character:`, `Location:`)
- [ ] Build a bidirectional relationship (Character ↔ Location) with resolvers
- [ ] Query 3 levels deep in Apollo Sandbox

<!-- > -->

## Resources

- [Apollo Server — Resolvers](https://www.apollographql.com/docs/apollo-server/data/resolvers/)
- [GraphQL — Execution](https://graphql.org/learn/execution/)
- [DataLoader (N+1 solution)](https://github.com/graphql/dataloader)
