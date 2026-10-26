# ACS 4330 Advanced - Week 6: Performance — The N+1 Problem and DataLoader

**Estimated time:** 3 hours for the lesson + 4–5 hours on your final project

**Prerequisites:** Completed Weeks 1–5.

This lesson is shorter on purpose. Use the extra time to get your final project feature-complete before deployment next week.

<!-- > -->

## Learning Outcomes

By the end of this lesson, you should be able to:

1. Explain the N+1 problem and why GraphQL nested resolvers cause it
2. Measure how many database queries a GraphQL request makes
3. Batch and cache lookups with DataLoader
4. Explain why DataLoaders are created once per request

<!-- > -->

## Review

From memory:

- When does a nested resolver like `Pet.shelter` run?
- What are the four arguments of a resolver?
- When does the context function run?

<!-- > -->

## The N+1 Problem

Run this query:

```graphql
query {
  pets {
    name
    shelter { name }
  }
}
```

How many database queries does it make?

1. `Query.pets` → `Pet.find()` — **1 query**
2. `Pet.shelter` runs **once per pet** → `Shelter.findById(...)` — **N queries**

With 50 pets, that's **51 queries**. Most of them fetch the same two or three shelters over and over.

This is the **N+1 problem**. It happens because each resolver runs independently: `Pet.shelter` for pet #7 doesn't know that pet #8 needs the same shelter.

<!-- > -->

## See It Happen

Turn on Mongoose's query logging. In `server.js`, after connecting:

```js
mongoose.set('debug', true)
```

Make sure your pets have shelters. Assign the seeded pets to your shelters in `mongosh` or Atlas, or run this once:

```js
// assign-shelters.js
import mongoose from 'mongoose'
import 'dotenv/config'
import Pet from './models/Pet.js'
import Shelter from './models/Shelter.js'

await mongoose.connect(process.env.MONGO_URI)
const shelters = await Shelter.find()
const pets = await Pet.find()
await Promise.all(
  pets.map((pet, i) => Pet.updateOne({ _id: pet._id }, { shelterId: shelters[i % shelters.length]._id }))
)
console.log(`Assigned ${pets.length} pets to ${shelters.length} shelters`)
await mongoose.disconnect()
```

Now run the query above and **count the lines in your terminal**. You'll see one `pets.find` and a long list of `shelters.findOne` calls.

<!-- > -->

## The Fix: Batching

What we want: collect every shelter ID that's needed for this request, then make **one** query:

```js
Shelter.find({ _id: { $in: [id1, id2, id3] } })
```

But each `Pet.shelter` resolver only sees one pet. Something has to collect the IDs across resolvers. That's what **DataLoader** does.

<!-- > -->

## How DataLoader Works

```js
const shelterLoader = new DataLoader(async (ids) => {
  // Called ONCE with every id requested in this tick
  // Must return results in the SAME ORDER as ids
})

shelterLoader.load('a')  // ┐
shelterLoader.load('b')  // ├─ batched into one call: ['a', 'b']
shelterLoader.load('a')  // ┘  'a' is only requested once (cached)
```

1. Each resolver calls `loader.load(id)` and gets back a promise
2. DataLoader **waits** until the current tick of the event loop finishes, collecting every ID
3. It calls your **batch function** once with all the unique IDs
4. Each promise resolves with its matching result

Two rules for the batch function:

- **Return an array the same length as `ids`**
- **Return results in the same order as `ids`.** MongoDB's `$in` doesn't promise any order, so you have to match results to IDs yourself

<!-- > -->

## Install

```bash
npm install dataloader
```

<!-- > -->

## Write the Loaders

`loaders.js`:

```js
import DataLoader from 'dataloader'

export function createLoaders({ Pet, Shelter }) {
  return {
    // id → Shelter
    shelterById: new DataLoader(async (ids) => {
      const shelters = await Shelter.find({ _id: { $in: ids } })
      const byId = new Map(shelters.map(s => [s.id, s]))
      return ids.map(id => byId.get(id) ?? null)
    }),

    // shelterId → [Pet]
    petsByShelterId: new DataLoader(async (shelterIds) => {
      const pets = await Pet.find({ shelterId: { $in: shelterIds } })
      const groups = new Map(shelterIds.map(id => [id, []]))
      for (const pet of pets) {
        groups.get(pet.shelterId.toString())?.push(pet)
      }
      return shelterIds.map(id => groups.get(id))
    }),
  }
}
```

The `Map` puts results back in the same order as the IDs. `byId.get(id) ?? null` handles IDs that don't exist: the result array still has one entry per ID.

<!-- > -->

## Use Them in Resolvers

```js
Pet: {
  shelter: (pet, _, { loaders }) =>
    pet.shelterId ? loaders.shelterById.load(pet.shelterId.toString()) : null,
},
Shelter: {
  pets: (shelter, _, { loaders }) => loaders.petsByShelterId.load(shelter.id),
},
```

**Always pass strings, not ObjectIds.** `pet.shelterId` is an ObjectId *object*, and DataLoader compares keys by identity (`===`). Two ObjectId objects for the same ID aren't `===`, so caching and de-duplication would silently fail. `.toString()` fixes this.

<!-- > -->

## Create Loaders Per Request

Add the loaders to **both** context functions in `server.js`:

```js
import { createLoaders } from './loaders.js'

// WebSocket context
context: async (ctx) => ({
  user: getUserFromToken(ctx.connectionParams?.authorization),
  ...models,
  loaders: createLoaders(models),
}),

// HTTP context
context: async ({ req }) => ({
  user: getUserFromToken(req.headers.authorization),
  ...models,
  loaders: createLoaders(models),
}),
```

**Why a new set of loaders on every request?** DataLoader caches results. A loader shared across all requests would:

- Serve **stale data**: a shelter renamed five minutes ago would still have its old name
- **Leak data between users**: one user's results could be served to another, which breaks your authorization rules

A loader per request caches only for the lifetime of one request. That's just long enough to remove duplicate lookups.

<!-- > -->

> **WebSocket caveat:** The WebSocket context runs once per *connection*, not per event, so those loaders live as long as the connection does. If your subscription payloads include nested fields that can change, create fresh loaders per event. Or return fully loaded data in the payload.

<!-- > -->

## Measure Again

Run the same query with `mongoose.set('debug', true)` still on.

| | Queries |
|---|---|
| Before | 1 + N (51 with 50 pets) |
| After | **2** — one for pets, one for all their shelters |

Try the reverse query:

```graphql
query {
  shelters {
    name
    pets { name }
  }
}
```

Before: 1 + number of shelters. After: 2.

Turn off `mongoose.set('debug', true)` when you're done. It's noisy.

<!-- > -->

## Other Performance Tools

You don't need these for this course, but you should know they exist:

| Tool | Problem it solves |
|---|---|
| **Query depth limits** (`graphql-depth-limit`) | Blocks deeply nested malicious queries like `pets { shelter { pets { shelter { ... } } } }` |
| **Pagination limits** | You did this in Week 4: cap `first`/`limit` |
| **`.lean()` in Mongoose** | Returns plain objects instead of full documents. Faster, but you lose the `id` virtual |
| **Persisted queries** | Clients send a short hash instead of the full query text |
| **HTTP caching / CDN** | Caches public query results at the edge |

<!-- > -->

## Challenges

<!-- > -->

**Challenge 1 — Measure the Problem**

1. Turn on `mongoose.set('debug', true)`
2. Assign shelters to all pets
3. Run `pets { name shelter { name } }` and record the number of queries in your README

<!-- > -->

**Challenge 2 — Fix It**

1. Add `loaders.js` with `shelterById` and `petsByShelterId`
2. Add `loaders` to both context functions
3. Update `Pet.shelter` and `Shelter.pets`
4. Measure again and record the new count in your README

<!-- > -->

**Challenge 3 — One More Loader**

Add a `userById` loader and use it in the `Pet.createdBy` resolver from Week 3. Verify that `pets { createdBy { email } }` makes 2 queries, not 1 + N.

<!-- > -->

**Challenge 4 — Stretch: Prime the Cache**

After `Shelter.find()` in the `shelters` query, call `loaders.shelterById.prime(shelter.id, shelter)` for each shelter. Then a later `shelter(id)` lookup in the same request doesn't hit the database. When is this useful?

<!-- > -->

## Final Project Milestone

1. Find every nested resolver in your project that queries the database
2. If you chose **DataLoader** as your Week 5/6 feature, add a loader for at least one of them and record before/after query counts in `PLAN.md`
3. Use the rest of the week to finish features. Next week is deployment: everything should work locally by the end of this week

<!-- > -->

## After This Lesson

- Push `pets/server/`
- Push your updated `final-project/`
- Mark Week 6 complete in the course tracker
- Bring a list of anything unfinished in your project to the check-in

<!-- > -->

### Evaluate Your Work

| | Does not meet expectations | Meets expectations | Exceeds expectations |
|:---:|:---:|:---:|:---:|
| Understanding | Can't explain N+1 | Explains why nested resolvers cause N+1 | Measured before/after query counts |
| DataLoader | Not implemented | One loader; batch function returns results in key order | Multiple loaders, including a one-to-many loader; string keys |
| Per-request loaders | Loaders shared globally | Loaders created in the context function | Can explain the stale-data and data-leak risks of shared loaders |

<!-- > -->

## Resources

- https://github.com/graphql/dataloader
- https://www.apollographql.com/docs/apollo-server/data/fetching-data#batching-and-caching
- https://mongoosejs.com/docs/api/mongoose.html#Mongoose.prototype.set()
