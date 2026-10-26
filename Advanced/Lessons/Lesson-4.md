# ACS 4330 Advanced - Week 4: Pagination

**Estimated time:** 4–5 hours for the lesson + 2–3 hours on your final project

**Prerequisites:** Completed Weeks 1–3.

<!-- > -->

## Learning Outcomes

By the end of this lesson, you should be able to:

1. Explain why APIs paginate list fields
2. Implement **offset** pagination with `limit`/`offset` and page controls in React
3. Implement **cursor** pagination with the Relay Connection pattern (`edges`, `node`, `cursor`, `pageInfo`)
4. Build "Load more" in React with `fetchMore` and a cache field policy
5. Choose between offset and cursor pagination for a given feature

<!-- > -->

## Review

From memory:

- What does `useQuery` return besides `data`, `loading`, and `error`?
- How do you pass variables to a query in `useQuery`?
- What do Mongoose's `.sort()` and `.limit()` do?

<!-- > -->

## Why Paginate?

Right now, `pets` returns **every** pet. With 10 pets that's fine. With 100,000, the response is huge, the database works hard, and the browser freezes rendering it all.

Pagination returns a list in chunks. The client asks for the next chunk when it needs one.

There are two common styles:

| | Offset | Cursor |
|---|---|---|
| Client asks for | "10 items, skipping the first 20" | "10 items after *this one*" |
| UI | Page numbers, Prev/Next | Load more, infinite scroll |
| Jump to page 7 | ✅ Easy | ❌ Not possible |
| Items added/removed while paging | ❌ Duplicates or skipped items | ✅ Stable |
| Large offsets | ❌ Slow — the database still walks past every skipped row | ✅ Fast with an index |

You'll build both.

<!-- > -->

## Seed Some Data

You need more than a handful of pets. Create `pets/server/seed.js`:

```js
import mongoose from 'mongoose'
import 'dotenv/config'
import Pet from './models/Pet.js'

const names = ['Biscuit', 'Mochi', 'Pepper', 'Luna', 'Ziggy', 'Olive', 'Bean', 'Rex', 'Nala', 'Pickles']
const species = ['cat', 'dog', 'bird', 'rabbit', 'other']

await mongoose.connect(process.env.MONGO_URI)

const pets = Array.from({ length: 50 }, (_, i) => ({
  name: `${names[i % names.length]} ${i + 1}`,
  species: species[i % species.length],
  age: (i % 15) + 1,
}))

await Pet.insertMany(pets)
console.log(`Inserted ${pets.length} pets`)
await mongoose.disconnect()
```

Run it once: `node seed.js`

<!-- > -->

# Part 1 — Offset Pagination

<!-- > -->

## Schema

Return the items together with information about the list:

```graphql
type PetPage {
  items: [Pet!]!
  total: Int!
  hasMore: Boolean!
}

type Query {
  petsPage(limit: Int = 10, offset: Int = 0): PetPage!
  # ...existing queries
}
```

- `items` — this page's pets
- `total` — how many pets exist in all (for "Page 2 of 5")
- `hasMore` — whether there's a next page

Keep the existing `pets` query for now. Adding a new field doesn't break existing clients.

<!-- > -->

## Resolver

```js
petsPage: async (_, { limit, offset }, { Pet }) => {
  const safeLimit = Math.min(Math.max(limit, 1), 50)  // between 1 and 50
  const [items, total] = await Promise.all([
    Pet.find().sort({ name: 1, _id: 1 }).skip(offset).limit(safeLimit),
    Pet.countDocuments(),
  ])
  return { items, total, hasMore: offset + items.length < total }
},
```

- **Always cap the limit.** Otherwise a client can ask for `limit: 1000000`.
- **Always sort.** Without a sort order, MongoDB can return items in any order, and pages could overlap. Adding `_id` as a tie-breaker keeps the order stable when names match.
- `Promise.all` runs both database calls at the same time.

<!-- > -->

## Try It

```graphql
query {
  petsPage(limit: 5, offset: 10) {
    total
    hasMore
    items { name }
  }
}
```

Change `offset` and watch the results move.

<!-- > -->

## React — Page Controls

Add to `src/queries.js`:

```js
export const GET_PETS_PAGE = gql`
  query GetPetsPage($limit: Int!, $offset: Int!) {
    petsPage(limit: $limit, offset: $offset) {
      total
      hasMore
      items { id name species }
    }
  }
`
```

`src/PetPages.jsx`:

```jsx
import { useState } from 'react'
import { useQuery } from '@apollo/client/react'
import { GET_PETS_PAGE } from './queries.js'

const PAGE_SIZE = 10

export default function PetPages() {
  const [page, setPage] = useState(0)
  const { data, previousData, loading, error } = useQuery(GET_PETS_PAGE, {
    variables: { limit: PAGE_SIZE, offset: page * PAGE_SIZE },
  })

  if (error) return <p>Error: {error.message}</p>
  const result = (data ?? previousData)?.petsPage
  if (!result) return <p>Loading...</p>

  const totalPages = Math.ceil(result.total / PAGE_SIZE)

  return (
    <section style={{ opacity: loading ? 0.5 : 1 }}>
      <ul>
        {result.items.map(pet => <li key={pet.id}>{pet.name}</li>)}
      </ul>
      <button onClick={() => setPage(p => p - 1)} disabled={page === 0}>Prev</button>
      <span> Page {page + 1} of {totalPages} </span>
      <button onClick={() => setPage(p => p + 1)} disabled={!result.hasMore}>Next</button>
    </section>
  )
}
```

New variables mean a new query. Apollo caches each page separately, so going back to a page you've already seen is instant.

`previousData` keeps the last page on screen while the next one loads, so the list doesn't flash to "Loading..." every time.

<!-- > -->

## The Offset Problem

Open page 2. In Sandbox (with your `Authorization` header — `addPet` requires login now), add a pet whose name sorts first (e.g. "Aardvark"). Click **Next**.

The last pet on page 2 shows up **again** at the top of page 3. Inserting a pet before your position pushed every item down one place. Deletes cause the opposite problem: items get skipped.

That's acceptable for an admin table. For a live feed, it isn't. Cursors fix it.

<!-- > -->

# Part 2 — Cursor Pagination (Relay Connections)

<!-- > -->

## The Connection Pattern

GitHub, Shopify, and most large GraphQL APIs use the **Relay Connection** shape:

```graphql
type PetEdge {
  node: Pet!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  endCursor: String
}

type PetConnection {
  edges: [PetEdge!]!
  pageInfo: PageInfo!
}

type Query {
  petsConnection(first: Int = 10, after: String): PetConnection!
  # ...existing queries
}
```

- **`node`** — the item itself
- **`cursor`** — an opaque bookmark for that item's position
- **`first`** — how many items to return
- **`after`** — "start after this cursor"
- **`pageInfo.endCursor`** — the cursor to pass as `after` to get the next page

The extra `edges` layer looks like overkill, but it's where you put information about the *relationship*, not the item. For example, GitHub puts `starredAt` on the edge of a user's starred repos.

<!-- > -->

## What's in a Cursor?

A cursor is an encoded position. We'll sort by `_id`, so a pet's `_id` marks its position. Encode it so clients treat it as opaque and don't build cursors themselves:

```js
const encodeCursor = (id) => Buffer.from(String(id)).toString('base64')
const decodeCursor = (cursor) => Buffer.from(cursor, 'base64').toString('utf8')
```

Because the cursor is opaque, you can change what's inside it later (e.g. to sort by date) without breaking clients.

<!-- > -->

## Resolver

```js
petsConnection: async (_, { first, after }, { Pet }) => {
  const limit = Math.min(Math.max(first, 1), 50)
  const filter = after ? { _id: { $gt: decodeCursor(after) } } : {}

  // Ask for one extra to find out if there's a next page
  const docs = await Pet.find(filter).sort({ _id: 1 }).limit(limit + 1)
  const hasNextPage = docs.length > limit
  const pets = docs.slice(0, limit)

  const edges = pets.map(pet => ({ node: pet, cursor: encodeCursor(pet.id) }))

  return {
    edges,
    pageInfo: {
      hasNextPage,
      endCursor: edges.length ? edges[edges.length - 1].cursor : null,
    },
  }
},
```

- `{ _id: { $gt: ... } }` means "IDs greater than the cursor." MongoDB uses the `_id` index to jump straight there. Unlike `skip`, it doesn't walk past earlier documents.
- Fetching `limit + 1` is a cheap way to calculate `hasNextPage` without a separate count.
- New pets get larger `_id`s, so they're added at the end. Pages you've already loaded don't shift.

<!-- > -->

## Try It

```graphql
query {
  petsConnection(first: 3) {
    edges {
      cursor
      node { name }
    }
    pageInfo { hasNextPage endCursor }
  }
}
```

Copy `endCursor` and pass it as `after`:

```graphql
query {
  petsConnection(first: 3, after: "PASTE_END_CURSOR") {
    edges { node { name } }
    pageInfo { hasNextPage endCursor }
  }
}
```

<!-- > -->

## React — "Load More"

Add to `src/queries.js`:

```js
export const GET_PETS_CONNECTION = gql`
  query GetPetsConnection($first: Int!, $after: String) {
    petsConnection(first: $first, after: $after) {
      edges {
        cursor
        node { id name species }
      }
      pageInfo { hasNextPage endCursor }
    }
  }
`
```

`src/PetFeed.jsx`:

```jsx
import { useQuery } from '@apollo/client/react'
import { GET_PETS_CONNECTION } from './queries.js'

export default function PetFeed() {
  const { data, loading, error, fetchMore } = useQuery(GET_PETS_CONNECTION, {
    variables: { first: 10 },
  })

  if (loading && !data) return <p>Loading...</p>
  if (error) return <p>Error: {error.message}</p>

  const { edges, pageInfo } = data.petsConnection

  return (
    <section>
      <ul>
        {edges.map(({ node }) => <li key={node.id}>{node.name}</li>)}
      </ul>
      {pageInfo.hasNextPage && (
        <button onClick={() => fetchMore({ variables: { after: pageInfo.endCursor } })}>
          Load more
        </button>
      )}
    </section>
  )
}
```

Click **Load more**. Instead of growing, the list is **replaced** by the next 10. You need to tell the cache how to merge pages.

<!-- > -->

## Merge Pages with a Field Policy

By default, Apollo stores each set of arguments as a separate cache entry. `first: 10` and `first: 10, after: "xyz"` are two different entries, so the second result replaces the first on screen.

A **field policy** tells the cache to treat every page as part of one list. Apollo ships one for Relay connections. In `src/apolloClient.js`:

```js
import { relayStylePagination } from '@apollo/client/utilities'

const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          petsConnection: relayStylePagination(),
        },
      },
    },
  }),
})
```

Now `fetchMore` appends. The list grows with each click.

> For offset pagination, Apollo has `offsetLimitPagination()`. You only need it if you want offset results to *append* (infinite scroll) instead of showing one page at a time.

<!-- > -->

## Pagination and Your Week 2 Cache Code

Your `addPet` `update` function changes the `pets` field. It doesn't touch `petsPage` or `petsConnection`, so new pets won't appear in the paginated lists.

The simplest fix is `refetchQueries: ['GetPetsConnection']` (or `'GetPetsPage'`). Inserting into a sorted, paginated list in the cache is possible, but it's tricky. Here, refetching is usually the right trade-off.

<!-- > -->

## Challenges

<!-- > -->

**Challenge 1 — Offset**

1. Seed 50 pets
2. Add `petsPage` with a capped limit and a stable sort
3. Build `PetPages` with Prev/Next and "Page X of Y"
4. Reproduce the duplicate-item problem by adding a pet while on page 2

<!-- > -->

**Challenge 2 — Cursor**

1. Add the connection types and `petsConnection`
2. Page through in Sandbox using `endCursor`
3. Build `PetFeed` with Load more and `relayStylePagination()`
4. Add a pet while you're partway through the feed. Confirm no duplicates

<!-- > -->

**Challenge 3 — Filtering**

Add a `species: String` argument to `petsConnection`. Include it in the MongoDB filter, and pass `relayStylePagination(['species'])` so each species keeps its own list in the cache. Add a species dropdown to `PetFeed`.

<!-- > -->

**Challenge 4 — Stretch: Sort by Name with Cursors**

Sorting by `_id` is easy because `_id` is unique. Make `petsConnection` sort by `name` instead. Hint: the cursor has to hold **both** `name` and `_id` (e.g. JSON-encode `{ name, id }`), and the filter becomes `name > cursorName OR (name == cursorName AND _id > cursorId)`.

<!-- > -->

## Final Project Milestone

1. Pick the list in your project most likely to grow large
2. Paginate it with offset **or** cursor pagination. Write one sentence in `PLAN.md` explaining your choice
3. Add the matching UI (page controls or Load more)

<!-- > -->

## After This Lesson

- Push `pets/server/` and `pets/client/`
- Push your updated `final-project/`
- Mark Week 4 complete in the course tracker

<!-- > -->

### Evaluate Your Work

| | Does not meet expectations | Meets expectations | Exceeds expectations |
|:---:|:---:|:---:|:---:|
| Offset pagination | Not implemented | `limit`/`offset` with `total` and `hasMore`; page controls work | Limit is capped; sort is stable; previous page stays visible while loading |
| Cursor pagination | Not implemented | Connection type with `edges`, `cursor`, and `pageInfo`; `after` works | Cursors are opaque; `limit + 1` trick used |
| Client cache | Load more replaces the list | `relayStylePagination` merges pages | Filter arguments set as `keyArgs`; new items handled with `refetchQueries` |
| Reasoning | Can't explain the difference | Explains when offset breaks | Pagination choice in the final project is justified |

<!-- > -->

## Resources

- https://graphql.org/learn/pagination/
- https://relay.dev/graphql/connections.htm — the Connection spec
- https://www.apollographql.com/docs/react/pagination/overview
- https://www.apollographql.com/docs/react/pagination/cursor-based
- https://docs.github.com/en/graphql/guides/using-pagination-in-the-graphql-api
