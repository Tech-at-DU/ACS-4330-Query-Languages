# ACS 4330 - Apollo Client Cache

**Estimated time:** 3–4 hours

**Prerequisites:** Completed Lessons 6 and 7. You have a React + Apollo Client app that runs queries and mutations.

<!-- > -->

## Learning Outcomes

By the end of this lesson, you should be able to:

1. Explain what the Apollo Client cache is and why it exists
2. Use `refetchQueries` to keep the UI in sync after a mutation
3. Use `cache.modify` to update the cache without a network request
4. Use `cache.evict` to remove items from the cache after a delete mutation

<!-- > -->

## Review

From memory:

- What does `useMutation` return?
- What happens when you call a mutation in Apollo Client — what does it send to the server?
- After a successful mutation, how does your UI currently update?

<!-- > -->

## 🗃️ What is the Apollo Client Cache?

Apollo Client stores every query result in an **in-memory cache**. When you run the same query twice, Apollo returns the cached result immediately instead of hitting the network again.

The cache is **normalized** — objects are stored by type + ID, not by query. If two queries both return `Pet:1`, Apollo stores it once and both queries share the same reference.

This is efficient but creates a problem: **when a mutation changes data on the server, the cache doesn't know.**

<!-- > -->

## The Problem

```jsx
function PetList() {
  const { data } = useQuery(GET_PETS)
  const [addPet] = useMutation(ADD_PET)

  return (
    <>
      {data?.pets.map(p => <p key={p.id}>{p.name}</p>)}
      <button onClick={() => addPet({ variables: { name: 'Snuffles', species: 'cat' } })}>
        Add Pet
      </button>
    </>
  )
}
```

When `addPet` runs:
1. Mutation fires — server adds the pet
2. Apollo cache still has the old `GET_PETS` result
3. UI shows the old list — new pet doesn't appear until you refresh

<!-- > -->

## Solution 1 — `refetchQueries` ♻️

Tell Apollo to re-run specific queries after a mutation succeeds.

```jsx
const [addPet] = useMutation(ADD_PET, {
  refetchQueries: ['GetPets'],
})
```

`'GetPets'` is the **operation name** from your query definition:

```js
const GET_PETS = gql`
  query GetPets {       # <-- operation name
    pets {
      id
      name
      species
    }
  }
`
```

After the mutation, Apollo automatically re-fetches `GetPets` and updates the UI.

<!-- > -->

**When to use `refetchQueries`:**

- Simple — one line of code
- Always correct — fetches fresh data from the server
- Costs an extra network request
- Best choice when simplicity matters more than performance

<!-- > -->

## Solution 2 — `cache.modify` ✏️

Update the cache directly after a mutation, without a second network request. More complex, but more efficient.

**Add a new item:**

```jsx
const [addPet] = useMutation(ADD_PET, {
  update(cache, { data: { addPet } }) {
    cache.modify({
      fields: {
        pets(existing = []) {
          const newRef = cache.writeFragment({
            data: addPet,
            fragment: gql`
              fragment NewPet on Pet {
                id
                name
                species
              }
            `
          })
          return [...existing, newRef]
        }
      }
    })
  }
})
```

What's happening:

1. `update(cache, result)` — Apollo calls this after the mutation succeeds
2. `cache.modify` — directly modifies a field in the cache
3. `cache.writeFragment` — turns the new pet object into a cache reference
4. Return the new array — Apollo updates all components using `pets`

<!-- > -->

**Update an existing item:**

If your mutation returns the updated object and it has an `id`, Apollo **automatically updates the cache** — no `update` function needed.

```jsx
const [updatePet] = useMutation(UPDATE_PET)
// As long as UPDATE_PET returns { id name species }, cache auto-updates
```

Apollo matches the returned `id` to the cached item and merges the fields.

<!-- > -->

## Solution 3 — `cache.evict` 🗑️

For delete mutations, remove the item from the cache entirely.

```jsx
const [deletePet] = useMutation(DELETE_PET, {
  update(cache, _, { variables: { id } }) {
    cache.evict({ id: cache.identify({ __typename: 'Pet', id }) })
    cache.gc()
  }
})
```

- `cache.identify` builds the cache key (`Pet:1`, `Pet:2`, etc.)
- `cache.evict` removes that item
- `cache.gc()` cleans up any dangling references

<!-- > -->

## Choosing an Approach

| Situation | Approach |
|---|---|
| Adding an item | `refetchQueries` or `cache.modify` + `writeFragment` |
| Updating an item | Automatic (if mutation returns updated object with `id`) |
| Deleting an item | `cache.evict` + `cache.gc()` |
| Multiple affected queries | `refetchQueries` with multiple query names |
| Performance matters | `cache.modify` — no extra network call |
| Simplicity matters | `refetchQueries` — always works |

<!-- > -->

## Challenge 1 — Verify the Problem

1. Open your React pet app with `useQuery` and `useMutation`
2. Add a pet via the UI
3. Verify the new pet does **not** appear until you manually refresh
4. Open the Apollo DevTools browser extension — inspect the cache before and after the mutation

<!-- > -->

## Challenge 2 — Fix with `refetchQueries`

Add `refetchQueries` to your `addPet` mutation:

```jsx
const [addPet] = useMutation(ADD_PET, {
  refetchQueries: ['GetPets'],
})
```

1. Make sure your `GET_PETS` query has an operation name (`query GetPets { ... }`)
2. Add a pet — it should appear immediately
3. Open Network tab in DevTools — confirm two requests fire (the mutation + the refetch)

<!-- > -->

## Challenge 3 — Fix Delete with `cache.evict`

Add a delete button to each pet. Use `cache.evict` so the pet disappears without a refetch:

```jsx
const [deletePet] = useMutation(DELETE_PET, {
  update(cache, _, { variables: { id } }) {
    cache.evict({ id: cache.identify({ __typename: 'Pet', id }) })
    cache.gc()
  }
})
```

Verify the pet disappears immediately and no extra network request fires.

<!-- > -->

## Challenge 4 — Apply to Your Final Project

For each mutation in your final project React client:

1. Identify what data changes after the mutation
2. Choose `refetchQueries` (simple) or `cache.modify`/`cache.evict` (efficient)
3. Implement and test — UI should update without a page refresh

<!-- > -->

## After This Lesson

- Submit your updated React client to GradeScope
- All mutations should update the UI without a manual refresh

<!-- > -->

### Evaluate Your Work

| | Does not meet expectations | Meets expectations | Exceeds expectations |
|:---:|:---:|:---:|:---:|
| Cache awareness | Can't explain why UI doesn't update after mutations | Identifies the cache as the cause; uses `refetchQueries` to fix it | Uses `cache.modify` or `cache.evict` for at least one mutation |
| `refetchQueries` | Not implemented | Correctly re-fetches after add/update mutations | Query operation names are consistent between query and mutation config |
| Cache manipulation | Not attempted | Uses `cache.evict` for delete | Uses `cache.modify` with `writeFragment` for add |

<!-- > -->

## Resources

- https://www.apollographql.com/docs/react/caching/overview
- https://www.apollographql.com/docs/react/caching/cache-interaction
- https://www.apollographql.com/docs/react/api/react/hooks#usemutation
