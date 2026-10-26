# ACS 4330 Advanced - Week 5: Real-Time with Subscriptions

**Estimated time:** 5 hours for the lesson + 2–3 hours on your final project

**Prerequisites:** Completed Weeks 1–4.

<!-- > -->

## Learning Outcomes

By the end of this lesson, you should be able to:

1. Explain how subscriptions differ from queries and mutations, and why they use WebSockets
2. Run Apollo Server with Express so it can serve HTTP and WebSockets together
3. Define `Subscription` fields and publish events from mutations with `PubSub`
4. Route subscriptions over WebSockets in Apollo Client with `ApolloLink.split`
5. Use `useSubscription` to show live data and update the cache

<!-- > -->

## Review

From memory:

- What does `cache.modify` + `writeFragment` do? When did you use it?
- What does `cache.evict` do?
- How does the server know who the user is on each request?

<!-- > -->

## Three Operation Types

| | Query | Mutation | Subscription |
|---|---|---|---|
| Purpose | Read data | Change data | Get **pushed** updates |
| Who starts it | Client | Client | Client subscribes; **server** sends events |
| How many responses | One | One | Many, until the client unsubscribes |
| Transport | HTTP | HTTP | **WebSocket** |

<!-- > -->

## Why WebSockets?

HTTP is request → response. The server can only answer; it can never start a conversation. To get updates over HTTP, the client has to keep asking: "Anything new?" every few seconds. That's called **polling**.

A **WebSocket** starts as an HTTP request, then "upgrades" to a connection that **stays open**. Either side can send a message at any time. When a pet is added, the server pushes it to every subscribed client immediately.

GraphQL subscriptions use the **`graphql-ws`** protocol on top of WebSockets. You won't write WebSocket code yourself; the libraries handle it.

<!-- > -->

## When Should You Use Subscriptions?

Subscriptions are for **small, frequent changes that users need to see right away**: chat messages, notifications, live scores, "someone else is editing this."

For everything else, refetching or polling is simpler. Apollo's `useQuery` supports `pollInterval: 5000`. Don't make everything live just because you can.

<!-- > -->

# Part 1 — The Server

<!-- > -->

## Why the Server Setup Changes

`startStandaloneServer` only speaks HTTP. To add a WebSocket server, you need access to the underlying Node HTTP server, so you'll switch to **Express**. This is the setup most production Apollo servers use.

The new structure:

1. Build one **schema** from `typeDefs` + `resolvers`
2. Create an Express app and an HTTP server
3. Attach a **WebSocket server** (for subscriptions) to the HTTP server
4. Attach **Apollo Server** (for queries and mutations) to Express at `/graphql`

Both use the same schema and the same port.

<!-- > -->

## Install

In `pets/server`:

```bash
npm install express cors @as-integrations/express5 ws graphql-ws @graphql-tools/schema graphql-subscriptions
```

| Package | Purpose |
|---|---|
| `express`, `cors` | HTTP server and cross-origin requests from your React app |
| `@as-integrations/express5` | Connects Apollo Server to Express 5 |
| `ws` | WebSocket server |
| `graphql-ws` | The GraphQL-over-WebSocket protocol |
| `@graphql-tools/schema` | `makeExecutableSchema` — one schema object shared by both servers |
| `graphql-subscriptions` | `PubSub` — an in-memory event bus |

<!-- > -->

## The New `server.js`

Replace your `server.js`:

```js
import { ApolloServer } from '@apollo/server'
import { ApolloServerPluginDrainHttpServer } from '@apollo/server/plugin/drainHttpServer'
import { expressMiddleware } from '@as-integrations/express5'
import { makeExecutableSchema } from '@graphql-tools/schema'
import { useServer } from 'graphql-ws/use/ws'
import { WebSocketServer } from 'ws'
import { createServer } from 'http'
import express from 'express'
import cors from 'cors'
import mongoose from 'mongoose'
import 'dotenv/config'
import typeDefs from './schema.js'
import resolvers from './resolvers.js'
import Pet from './models/Pet.js'
import Shelter from './models/Shelter.js'
import User from './models/User.js'
import { getUserFromToken } from './auth.js'

await mongoose.connect(process.env.MONGO_URI)
console.log('Connected to MongoDB')

const schema = makeExecutableSchema({ typeDefs, resolvers })
const models = { Pet, Shelter, User }

const app = express()
const httpServer = createServer(app)

// WebSocket server for subscriptions
const wsServer = new WebSocketServer({ server: httpServer, path: '/graphql' })
const wsCleanup = useServer(
  {
    schema,
    // Runs once per WebSocket connection
    context: async (ctx) => ({
      user: getUserFromToken(ctx.connectionParams?.authorization),
      ...models,
    }),
  },
  wsServer
)
```

<!-- > -->

```js
// Apollo Server for queries and mutations
const server = new ApolloServer({
  schema,
  plugins: [
    // Shut down the HTTP server cleanly
    ApolloServerPluginDrainHttpServer({ httpServer }),
    // Shut down the WebSocket server cleanly
    {
      async serverWillStart() {
        return {
          async drainServer() {
            await wsCleanup.dispose()
          },
        }
      },
    },
  ],
})

await server.start()

app.use(
  '/graphql',
  cors(),
  express.json(),
  expressMiddleware(server, {
    // Runs once per HTTP request
    context: async ({ req }) => ({
      user: getUserFromToken(req.headers.authorization),
      ...models,
    }),
  })
)

const PORT = process.env.PORT || 4000
httpServer.listen(PORT, () => {
  console.log(`Server ready at http://localhost:${PORT}/graphql`)
})
```

<!-- > -->

## What Changed

- **The URL is now `http://localhost:4000/graphql`**, not `http://localhost:4000/`. Update `HttpLink` in your client.
- **There are two context functions.** HTTP requests carry headers. A WebSocket connection carries `connectionParams`, which the client sends once when it connects. Both build the same context shape.
- **`PORT` comes from the environment**, which you'll need for deployment in Week 7.
- Your resolvers, schema, models, and `auth.js` **don't change**.

Restart the server and confirm your existing queries and mutations still work in Sandbox at http://localhost:4000/graphql.

<!-- > -->

## Create the Event Bus

`pubsub.js`:

```js
import { PubSub } from 'graphql-subscriptions'

export const pubsub = new PubSub()

export const PET_ADDED = 'PET_ADDED'
export const PET_DELETED = 'PET_DELETED'
```

`PubSub` is a simple event bus: `publish` sends an event, and every subscriber listening for that event name receives it. Using constants for event names means a typo shows up as an error instead of a subscription that silently never fires.

<!-- > -->

## Add Subscriptions to the Schema

In `schema.js`:

```graphql
type Subscription {
  petAdded: Pet!
  petDeleted: ID!
}
```

<!-- > -->

## Publish from Mutations

In `resolvers.js`:

```js
import { pubsub, PET_ADDED, PET_DELETED } from './pubsub.js'

// ...in Mutation
addPet: async (_, args, context) => {
  const user = requireAuth(context)
  const pet = await context.Pet.create({ ...args, createdBy: user.userId })
  pubsub.publish(PET_ADDED, { petAdded: pet })
  return pet
},

deletePet: async (_, { id }, context) => {
  const pet = await context.Pet.findById(id)
  if (!pet) return false
  requireOwnerOrAdmin(context, pet)
  await pet.deleteOne()
  pubsub.publish(PET_DELETED, { petDeleted: id })
  return true
},
```

The payload's key (`petAdded`) **must match the subscription field name**. GraphQL uses it to find the value to send.

<!-- > -->

## Subscription Resolvers

Subscription resolvers have a `subscribe` function instead of a plain function:

```js
const resolvers = {
  Query: { /* ... */ },
  Mutation: { /* ... */ },
  Subscription: {
    petAdded: {
      subscribe: () => pubsub.asyncIterableIterator(PET_ADDED),
    },
    petDeleted: {
      subscribe: () => pubsub.asyncIterableIterator(PET_DELETED),
    },
  },
  Pet: { /* ... */ },
  Shelter: { /* ... */ },
}
```

`subscribe` returns an **async iterator**, a stream of events. Each published event flows through the normal `Pet` resolvers, so nested fields like `shelter` work in subscriptions too.

> **Older examples** use `pubsub.asyncIterator(...)`. It was renamed to `asyncIterableIterator` in `graphql-subscriptions` v3.

<!-- > -->

## Test in Sandbox

1. Open http://localhost:4000/graphql in **two** browser tabs
2. In tab 1, run:

   ```graphql
   subscription {
     petAdded {
       name
       species
       shelter { name }
     }
   }
   ```

   Sandbox shows it's listening.

3. In tab 2, add the `Authorization` header and run an `addPet` mutation
4. Watch the pet appear in tab 1. Add a few more

<!-- > -->

## Only In-Memory

`PubSub` keeps subscribers in the server's memory. That's fine for one server. If you run **two** server instances, a mutation on instance A never reaches subscribers on instance B. Production apps swap in a shared bus like Redis (`graphql-redis-subscriptions`), and the resolver code stays the same.

<!-- > -->

# Part 2 — The Client

<!-- > -->

## Install

In `pets/client`:

```bash
npm install graphql-ws
```

<!-- > -->

## Split the Link

Queries and mutations go over HTTP. Subscriptions go over the WebSocket. `ApolloLink.split` sends each operation the right way.

`src/apolloClient.js`:

```js
import { ApolloClient, ApolloLink, HttpLink, InMemoryCache } from '@apollo/client'
import { SetContextLink } from '@apollo/client/link/context'
import { GraphQLWsLink } from '@apollo/client/link/subscriptions'
import { relayStylePagination } from '@apollo/client/utilities'
import { createClient } from 'graphql-ws'
import { OperationTypeNode } from 'graphql'

const authHeader = () => {
  const token = localStorage.getItem('token')
  return token ? `Bearer ${token}` : ''
}

const httpLink = new HttpLink({ uri: 'http://localhost:4000/graphql' })

const authLink = new SetContextLink(({ headers }) => ({
  headers: { ...headers, authorization: authHeader() },
}))

const wsLink = new GraphQLWsLink(
  createClient({
    url: 'ws://localhost:4000/graphql',
    connectionParams: () => ({ authorization: authHeader() }),
  })
)

const splitLink = ApolloLink.split(
  ({ operationType }) => operationType === OperationTypeNode.SUBSCRIPTION,
  wsLink,                    // true  → WebSocket
  authLink.concat(httpLink)  // false → HTTP with auth header
)
```

<!-- > -->

```js
const client = new ApolloClient({
  link: splitLink,
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

export default client
```

Note the URLs: `http://` for HTTP and `ws://` for WebSockets. In production with HTTPS, they become `https://` and `wss://`.

<!-- > -->

## A Live Activity Feed

Add to `src/queries.js`:

```js
export const PET_ADDED = gql`
  subscription OnPetAdded {
    petAdded { id name species }
  }
`

export const PET_DELETED = gql`
  subscription OnPetDeleted {
    petDeleted
  }
`
```

`src/LiveActivity.jsx`:

```jsx
import { useSubscription } from '@apollo/client/react'
import { PET_ADDED } from './queries.js'

export default function LiveActivity() {
  const { data, loading, error } = useSubscription(PET_ADDED)

  if (error) return <p>Live updates unavailable: {error.message}</p>
  if (loading || !data) return <p>Listening for new pets…</p>
  return <p>🐾 Just added: {data.petAdded.name} the {data.petAdded.species}</p>
}
```

`loading` is `true` until the first event arrives. After that, `data` always holds the **latest** event.

Open the app in **two** windows. Add a pet in one and watch the other.

<!-- > -->

## Update the List When Events Arrive

The activity feed shows the event, but the pet list in the other window doesn't change. Use `onData` to write each event into the cache, the same way you did in Week 2:

```jsx
import { gql } from '@apollo/client'
import { useSubscription } from '@apollo/client/react'
import { PET_ADDED, PET_DELETED } from './queries.js'

export function usePetSubscriptions() {
  useSubscription(PET_ADDED, {
    onData: ({ client, data }) => {
      const pet = data.data?.petAdded
      if (!pet) return
      client.cache.modify({
        fields: {
          pets(existing = [], { readField }) {
            // This window may have added the pet itself
            if (existing.some(ref => readField('id', ref) === pet.id)) return existing
            const ref = client.cache.writeFragment({
              data: pet,
              fragment: gql`fragment LivePet on Pet { id name species }`,
            })
            return [...existing, ref]
          },
        },
      })
    },
  })
```

<!-- > -->

```jsx
  useSubscription(PET_DELETED, {
    onData: ({ client, data }) => {
      const id = data.data?.petDeleted
      if (!id) return
      client.cache.evict({ id: client.cache.identify({ __typename: 'Pet', id }) })
      client.cache.gc()
    },
  })
}
```

Call `usePetSubscriptions()` at the top of `App`.

**Why the duplicate check?** The window that ran `addPet` already added the pet with its mutation `update` function. Then it receives its own `petAdded` event too. Without the check, the pet would show up twice.

<!-- > -->

## Auth and WebSockets

`connectionParams` is sent **once**, when the WebSocket connects. If the user logs in afterwards, the open connection still has the old (empty) token.

Our subscriptions are public, so this doesn't matter here. If your subscriptions require login, restart the connection after login and logout. Keep a reference to the `graphql-ws` client and call `wsClient.terminate()`. It reconnects automatically with fresh `connectionParams`.

<!-- > -->

## Challenges

<!-- > -->

**Challenge 1 — Server**

1. Switch `server.js` to Express + WebSockets. Confirm queries and mutations still work at `/graphql`
2. Add `pubsub.js`, the `Subscription` type, and the resolvers
3. Test `petAdded` and `petDeleted` with two Sandbox tabs

<!-- > -->

**Challenge 2 — Client**

1. Update `HttpLink` to `/graphql` and add the WebSocket link with `ApolloLink.split`
2. Add `LiveActivity`
3. Add `usePetSubscriptions`. With two windows open, adds and deletes in one appear in the other with no refresh
4. In DevTools → Network → **WS**, click the WebSocket connection and watch the messages

<!-- > -->

**Challenge 3 — Filtered Subscription**

Let clients subscribe to a single species: `petAdded(species: String): Pet!`. Use `withFilter` from `graphql-subscriptions`:

```js
import { withFilter } from 'graphql-subscriptions'

petAdded: {
  subscribe: withFilter(
    () => pubsub.asyncIterableIterator(PET_ADDED),
    (payload, variables) => !variables.species || payload.petAdded.species === variables.species
  ),
},
```

Add a species picker to `LiveActivity`.

<!-- > -->

**Challenge 4 — Stretch: Protected Subscription**

Add `myPetUpdated: Pet!`, which only fires for pets the subscriber created. Use `withFilter` and compare `payload.myPetUpdated.createdBy?.toString()` with `context.user?.userId` (the filter function's third argument is context). Publish it from `updatePet`.

<!-- > -->

## Final Project Milestone

Choose **one** for your project:

- **Subscriptions** (this week): add at least one subscription that makes sense for your app, and show it live in the client, **or**
- **DataLoader** (next week): you'll decide after Week 6

Either way, switch your project server to the Express setup this week. You'll need `/graphql` and `process.env.PORT` to deploy.

<!-- > -->

## After This Lesson

- Push `pets/server/` and `pets/client/`
- Push your updated `final-project/`
- Mark Week 5 complete in the course tracker

<!-- > -->

### Evaluate Your Work

| | Does not meet expectations | Meets expectations | Exceeds expectations |
|:---:|:---:|:---:|:---:|
| Server setup | Still on `startStandaloneServer`, or queries broke | Express + WebSocket server; queries, mutations, and subscriptions all work | Both context functions build the same shape; clean shutdown plugins in place |
| Publishing | No events published | Mutations publish; payload key matches field name | Event names are constants; uses `withFilter` |
| Client | Subscriptions don't connect | `ApolloLink.split` routes subscriptions over WebSocket; `useSubscription` shows events | Events update the cache without duplicates |
| Judgment | — | Can explain HTTP vs. WebSocket | Can explain when polling is the better choice |

<!-- > -->

## Resources

- https://www.apollographql.com/docs/apollo-server/data/subscriptions
- https://www.apollographql.com/docs/react/data/subscriptions
- https://the-guild.dev/graphql/ws — `graphql-ws`
- https://github.com/apollographql/graphql-subscriptions
- https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API
