# ACS 4330 Advanced - Week 2: Apollo Client Cache and Optimistic UI

**Estimated time:** 4 hours for the lesson + 2–3 hours on your final project

**Prerequisites:** Completed Week 1. Your Pets server runs on MongoDB with `Pet` and `Shelter` types.

> **Already did "Apollo Client Cache" (Lesson 11) in ACS 4330?** Skim Part 1. Part 2 (optimistic UI) is new.

<!-- > -->

## Learning Outcomes

By the end of this lesson, you should be able to:

1. Explain what the Apollo Client cache is and why it can show stale data
2. Use `refetchQueries` to keep the UI in sync after a mutation
3. Use `cache.modify` and `cache.writeFragment` to add an item to the cache without a network request
4. Use `cache.evict` to remove an item from the cache after a delete
5. Use `optimisticResponse` to update the UI before the server responds

<!-- > -->

## Review

From memory:

- What does `useMutation` return?
- What does a mutation send to the server? What does it get back?
- After a successful mutation in your ACS 4330 project, how did the UI update?

<!-- > -->

## Setup — The Pets Client

Skip this if you already have a React + Apollo Client app that talks to your Pets server.

```bash
cd pets
npm create vite@latest client -- --template react
cd client
npm install
npm install @apollo/client graphql rxjs
```

<!-- > -->

`src/apolloClient.js`:

```js
import { ApolloClient, HttpLink, InMemoryCache } from '@apollo/client'

const client = new ApolloClient({
  link: new HttpLink({ uri: 'http://localhost:4000/' }),
  cache: new InMemoryCache(),
})

export default client
```

`src/main.jsx`:

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { ApolloProvider } from '@apollo/client/react'
import client from './apolloClient.js'
import App from './App.jsx'

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <ApolloProvider client={client}>
      <App />
    </ApolloProvider>
  </StrictMode>
)
```

<!-- > -->

`src/queries.js` — keep all your GraphQL operations in one file:

```js
import { gql } from '@apollo/client'

export const GET_PETS = gql`
  query GetPets {
    pets {
      id
      name
      species
    }
  }
`

export const ADD_PET = gql`
  mutation AddPet($name: String!, $species: String!) {
    addPet(name: $name, species: $species) {
      id
      name
      species
    }
  }
`

export const DELETE_PET = gql`
  mutation DeletePet($id: ID!) {
    deletePet(id: $id)
  }
`
```

<!-- > -->

`src/App.jsx`:

```jsx
import { useState } from 'react'
import { useQuery, useMutation } from '@apollo/client/react'
import { GET_PETS, ADD_PET, DELETE_PET } from './queries.js'

export default function App() {
  const { data, loading, error } = useQuery(GET_PETS)
  const [addPet] = useMutation(ADD_PET)
  const [deletePet] = useMutation(DELETE_PET)
  const [name, setName] = useState('')
  const [species, setSpecies] = useState('cat')

  const handleSubmit = async (e) => {
    e.preventDefault()
    await addPet({ variables: { name, species } })
    setName('')
  }

  if (loading) return <p>Loading...</p>
  if (error) return <p>Error: {error.message}</p>

  return (
    <main>
      <h1>Pets</h1>
      <form onSubmit={handleSubmit}>
        <input value={name} onChange={e => setName(e.target.value)} placeholder="Name" required />
        <select value={species} onChange={e => setSpecies(e.target.value)}>
          <option>cat</option>
          <option>dog</option>
          <option>bird</option>
          <option>rabbit</option>
          <option>other</option>
        </select>
        <button>Add Pet</button>
      </form>
      <ul>
        {data.pets.map(pet => (
          <li key={pet.id}>
            {pet.name} ({pet.species})
            <button onClick={() => deletePet({ variables: { id: pet.id } })}>Delete</button>
          </li>
        ))}
      </ul>
    </main>
  )
}
```

Start the server (`npm run dev` in `pets/server`) and the client (`npm run dev` in `pets/client`).

> **Apollo Client 4:** React hooks and `ApolloProvider` are imported from `@apollo/client/react`. `gql`, `ApolloClient`, `InMemoryCache`, and links are imported from `@apollo/client`. If you see older examples importing `useQuery` from `@apollo/client`, they're written for Apollo Client 3.

<!-- > -->

## Install Apollo DevTools

Install the [Apollo Client DevTools](https://www.apollographql.com/docs/react/development-testing/developer-tooling#apollo-client-devtools) browser extension. It adds an **Apollo** tab to your browser's developer tools where you can inspect the cache. You'll use it throughout this lesson.

<!-- > -->

# Part 1 — Keeping the Cache in Sync

<!-- > -->

## 🗃️ What is the Apollo Client Cache?

Apollo Client stores every query result in an **in-memory cache**. When you run the same query twice, Apollo returns the cached result instead of hitting the network again.

The cache is **normalized**: objects are stored by type + ID, not by query. If two queries both return the pet with ID `abc`, Apollo stores it once as `Pet:abc`, and both queries point to that one entry.

This is efficient, but it creates a problem: **when a mutation changes data on the server, the cache doesn't know which lists to change.**

<!-- > -->

## The Problem

Add a pet in the app. Does it appear?

When `addPet` runs:

1. The mutation fires, and the server adds the pet
2. Apollo puts the new `Pet` object in the cache, but nothing tells it to add that pet to the `pets` list
3. The UI still shows the old list. The new pet only appears after you refresh the page

Delete has the same problem: the pet is gone from the server, but the cached `pets` list still points to it.

<!-- > -->

## Solution 1 — `refetchQueries` ♻️

Tell Apollo to re-run specific queries after a mutation succeeds:

```jsx
const [addPet] = useMutation(ADD_PET, {
  refetchQueries: ['GetPets'],
})
```

`'GetPets'` is the **operation name** from your query definition (`query GetPets { ... }`). Always name your operations.

**When to use `refetchQueries`:**

- ✅ Simple — one line of code
- ✅ Always correct — gets fresh data from the server
- ❌ Costs an extra network request
- Best when simplicity matters more than speed

<!-- > -->

## Solution 2 — Return the `id` (Updates Are Free) 🆓

If a mutation returns the changed object **with its `id`**, Apollo finds the cached object by type + ID and merges the new fields in. Every component showing that pet re-renders.

```graphql
mutation UpdatePet($id: ID!, $name: String) {
  updatePet(id: $id, name: $name) {
    id      # ← required for the automatic update
    name
  }
}
```

No `update` function or `refetchQueries` needed. **Updates to existing items take care of themselves.** Only adds and deletes change *which* items are in a list, and those need your help.

<!-- > -->

## Solution 3 — `cache.modify` + `writeFragment` ✏️

Add the new item to the cached list directly, without a second network request:

```jsx
const [addPet] = useMutation(ADD_PET, {
  update(cache, { data }) {
    cache.modify({
      fields: {
        pets(existing = []) {
          const newRef = cache.writeFragment({
            data: data.addPet,
            fragment: gql`
              fragment NewPet on Pet {
                id
                name
                species
              }
            `,
          })
          return [...existing, newRef]
        },
      },
    })
  },
})
```

What's happening:

1. `update(cache, result)` — Apollo calls this after the mutation succeeds
2. `cache.modify` — changes a field in the cache; here, the root `pets` field
3. `cache.writeFragment` — saves the new pet and returns a **reference** to it (like `{ __ref: 'Pet:abc' }`)
4. Return the new array — every component using `pets` re-renders

Import `gql` from `@apollo/client`.

<!-- > -->

## Solution 4 — `cache.evict` 🗑️

For deletes, remove the item from the cache entirely:

```jsx
const [deletePet] = useMutation(DELETE_PET, {
  update(cache, _result, { variables }) {
    cache.evict({ id: cache.identify({ __typename: 'Pet', id: variables.id }) })
    cache.gc()
  },
})
```

- `cache.identify` builds the cache key (`Pet:abc`)
- `cache.evict` removes that object
- Lists that pointed to it now have a dangling reference. Apollo filters those out of lists automatically
- `cache.gc()` cleans up anything no longer reachable

<!-- > -->

## Choosing an Approach

| Situation | Approach |
|---|---|
| Updating an item | Automatic — return the `id` and changed fields |
| Adding an item | `refetchQueries` (simple) or `cache.modify` + `writeFragment` (no extra request) |
| Deleting an item | `cache.evict` + `cache.gc()` |
| Many queries affected, or sorted/filtered lists | `refetchQueries` — let the server work out the new lists |

<!-- > -->

# Part 2 — Optimistic UI

<!-- > -->

## ⚡ Why Optimistic UI?

Even with `cache.modify`, the UI waits for the server: click → request → response → update. On a slow connection, that's a noticeable delay.

Most mutations succeed. **Optimistic UI** assumes success: Apollo shows the expected result right away, then replaces it with the real response when it arrives. If the mutation fails, Apollo throws away the optimistic result.

<!-- > -->

## Simulate a Slow Network

To see the difference, slow the server down. Add this at the top of your `addPet` and `deletePet` resolvers **temporarily**:

```js
await new Promise(resolve => setTimeout(resolve, 1500))
```

(The resolver needs to be `async` for `await` to work.)

Add a pet. Notice the 1.5-second delay before it appears.

<!-- > -->

## Optimistic Add

Pass `optimisticResponse` when you call the mutation. It must have the **same shape as the real response**, including `__typename`:

```jsx
const handleSubmit = async (e) => {
  e.preventDefault()
  await addPet({
    variables: { name, species },
    optimisticResponse: {
      addPet: {
        __typename: 'Pet',
        id: `temp-${Date.now()}`,
        name,
        species,
      },
    },
  })
  setName('')
}
```

Keep the `update` function from Solution 3. Now the pet appears instantly.

<!-- > -->

## What Actually Happens

1. Apollo calls `update` with the **optimistic** response. The temporary pet is written to a separate optimistic layer of the cache, and the UI shows it immediately.
2. The real request goes to the server.
3. When the response arrives, Apollo **throws away** the optimistic layer and calls `update` again with the **real** response, which has the real `id`.
4. If the request fails, Apollo throws away the optimistic layer, and the temporary pet disappears.

Because the optimistic layer is thrown away, the pet doesn't show up twice. Your `update` function doesn't need to know which kind of response it got.

<!-- > -->

## Optimistic Delete

```jsx
<button
  onClick={() =>
    deletePet({
      variables: { id: pet.id },
      optimisticResponse: { deletePet: true },
    })
  }
>
  Delete
</button>
```

Keep the `cache.evict` `update` function from Solution 4. The pet disappears immediately.

<!-- > -->

## Show Pending Items

Optimistic items have the temporary ID you made up, so you can style them differently:

```jsx
<li key={pet.id} style={{ opacity: pet.id.startsWith('temp-') ? 0.5 : 1 }}>
```

The pet fades in at half opacity, then becomes solid when the server confirms it.

<!-- > -->

## Handle Failure

Make the server fail on purpose. In `addPet`, temporarily add:

```js
throw new Error('Server is having a bad day')
```

Add a pet: it appears, then disappears. The user needs to know why, so catch the error:

```jsx
const [addPet, { error: addError }] = useMutation(ADD_PET, { /* ... */ })
```

```jsx
const handleSubmit = async (e) => {
  e.preventDefault()
  try {
    await addPet({ /* ... */ })
    setName('')
  } catch {
    // addError is set; the optimistic pet has been removed
  }
}
```

```jsx
{addError && <p role="alert">Could not add pet: {addError.message}</p>}
```

The mutate function **rejects** when the server returns an error. Without `try`/`catch`, you get an unhandled promise rejection in the console.

Remove the `throw` and the `setTimeout` delays when you're done.

<!-- > -->

## Challenges

<!-- > -->

**Challenge 1 — See the Problem**

1. Set up the Pets client (or use your own)
2. Add a pet. Confirm it does **not** appear until you refresh
3. Open the Apollo DevTools **Cache** tab. Find the new `Pet:...` entry, and notice that `ROOT_QUERY.pets` doesn't include it

<!-- > -->

**Challenge 2 — `refetchQueries`, then `cache.modify`**

1. Fix `addPet` with `refetchQueries`. In the Network tab, confirm **two** requests fire
2. Replace it with `cache.modify` + `writeFragment`. Confirm only **one** request fires

<!-- > -->

**Challenge 3 — Delete and Update**

1. Fix `deletePet` with `cache.evict` + `cache.gc()`
2. Add an inline "rename" input that calls `updatePet`. Confirm the name updates without any `update` function

<!-- > -->

**Challenge 4 — Optimistic UI**

1. Add the 1.5-second delay to the server
2. Make add and delete optimistic
3. Fade in pending pets
4. Force an error and show the error message in the UI
5. Remove the delay and the forced error

<!-- > -->

## Final Project Milestone

Your project needs a React client by the end of this week. Reuse your ACS 4330 client, or set one up as shown above.

For **every** mutation in your project:

1. Decide what changes in the cache: an update, an add, or a delete
2. Use the matching approach from "Choosing an Approach"
3. Make at least one mutation optimistic
4. Test it: the UI updates without a page refresh

<!-- > -->

## After This Lesson

- Push `pets/client/` to your course repo
- Push your updated `final-project/client/`
- Mark Week 2 complete in the course tracker

<!-- > -->

### Evaluate Your Work

| | Does not meet expectations | Meets expectations | Exceeds expectations |
|:---:|:---:|:---:|:---:|
| Cache awareness | Can't explain why the UI doesn't update after a mutation | Explains normalization; uses `refetchQueries` correctly | Chooses between refetch and cache writes for good reasons |
| Cache writes | Not attempted | `cache.evict` for delete | `cache.modify` + `writeFragment` for add; updates rely on `id` |
| Optimistic UI | Not attempted | One mutation is optimistic | Pending items are styled; failures show an error message |

<!-- > -->

## Resources

- https://www.apollographql.com/docs/react/caching/overview
- https://www.apollographql.com/docs/react/caching/cache-interaction
- https://www.apollographql.com/docs/react/performance/optimistic-ui
- https://www.apollographql.com/docs/react/data/mutations
