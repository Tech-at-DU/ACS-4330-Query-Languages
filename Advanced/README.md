# ACS 4330 - Advanced GraphQL

This course picks up where ACS 4330 Query Languages left off. Students have built a full-stack GraphQL application with Apollo Server, React + Apollo Client, and a final project. This term covers the topics that make GraphQL production-ready: real databases, authentication, real-time data, pagination, performance, and deployment.

**Course Delivery**: Independent study | 7 weeks | 75 total hours (~10 hours/week)

**Course Credits**: 3 units

## Prerequisites

- Completed ACS 4330 Query Languages (Lessons 1–9 + Final Project)
- Comfortable with: GraphQL schemas, resolvers, mutations, Apollo Server 4, React + Apollo Client, `useQuery`, `useMutation`
- Node.js 18+, basic async/await, ES modules

## Learning Objectives

By the end of this course you will be able to:

1. Persist GraphQL data with MongoDB and Mongoose
2. Manage Apollo Client cache efficiently — updates, deletes, optimistic UI
3. Implement JWT authentication and role-based authorization
4. Paginate results using both offset and cursor-based (Relay-style) patterns
5. Build real-time features with GraphQL subscriptions
6. Solve the N+1 problem with DataLoader
7. Design schemas with unions, interfaces, and custom scalars
8. Test GraphQL resolvers and mutations
9. Deploy a full-stack GraphQL app to the internet

---

## Schedule

**Format:** Independent study. Work through two lessons per week on your own. Meet with your instructor once per week.

| Week | Lessons | Topics | Graded |
|:-----|:--------|:-------|:-------|
| 1 | [Lesson 1] + [Lesson 2] | MongoDB + Mongoose, data modeling | L1 server |
| 2 | [Lesson 3] + [Lesson 4] | Apollo cache strategies, optimistic UI | — |
| 3 | [Lesson 5] + [Lesson 6] | JWT auth, authorization + roles | L5 auth server |
| 4 | [Lesson 7] + [Lesson 8] | Offset pagination, cursor/Relay pagination | — |
| 5 | [Lesson 9] + [Lesson 10] | Subscriptions setup, `useSubscription` in React | — |
| 6 | [Lesson 11] + [Lesson 12] | N+1 + DataLoader, unions + interfaces | — |
| 7 | [Lesson 13] + presentations | Testing + deployment, final project demo | Final project, final assessment |

[Lesson 1]: Lessons/Lesson-1.md
[Lesson 2]: Lessons/Lesson-2.md
[Lesson 3]: Lessons/Lesson-3.md
[Lesson 4]: Lessons/Lesson-4.md
[Lesson 5]: Lessons/Lesson-5.md
[Lesson 6]: Lessons/Lesson-6.md
[Lesson 7]: Lessons/Lesson-7.md
[Lesson 8]: Lessons/Lesson-8.md
[Lesson 9]: Lessons/Lesson-9.md
[Lesson 10]: Lessons/Lesson-10.md
[Lesson 11]: Lessons/Lesson-11.md
[Lesson 12]: Lessons/Lesson-12.md
[Lesson 13]: Lessons/Lesson-13.md
[Final Project]: Assignments/FinalProjectSpec.md

---

## Lesson Outline

### Week 1 — Real Persistence

**Lesson 1 — MongoDB + Mongoose**
Connect a GraphQL API to a real database. Install Mongoose, define models, and replace in-memory arrays with `find`, `findById`, `create`, `findByIdAndUpdate`, and `findByIdAndDelete`. Understand the difference between a Mongoose schema and a GraphQL schema. Pass models through context so resolvers stay clean. Cover the `_id` → `id` virtual so GraphQL IDs work correctly.

**Lesson 2 — Data Modeling with Mongoose**
Go deeper on data relationships in MongoDB. Compare embedding documents vs. storing references (IDs). Use Mongoose `populate()` to resolve references in a nested resolver. Add indexes for fields you query often. Design a schema with at least two related types — for example, `Post` and `Author`, or `Product` and `Category`. Explore schema validation options (required, min/max, enum).

---

### Week 2 — Apollo Client Cache

**Lesson 3 — Apollo Cache Strategies**
Understand why stale cache causes bugs after mutations. Learn three strategies for keeping the cache in sync: (1) `refetchQueries` — simple but fetches from the network; (2) returning `id` in a mutation so Apollo auto-updates the cached item; (3) `cache.modify` + `writeFragment` — direct cache writes for adding new items without a network round-trip. Know when to use each.

**Lesson 4 — Optimistic UI and Cache Eviction**
Make mutations feel instant with `optimisticResponse`. Show the new item in the list before the server replies — then reconcile when the real response arrives. Handle the delete case: `cache.evict` removes an item by cache ID, and `cache.gc()` cleans up dangling references. Build a UI where add, update, and delete all feel immediate.

---

### Week 3 — Authentication and Authorization

**Lesson 5 — JWT Authentication**
Implement user registration and login as GraphQL mutations. Hash passwords with `bcrypt`, sign tokens with `jsonwebtoken`, and return an `AuthPayload` containing the token and user. Verify the token in Apollo Server's `context` function so every resolver can check `context.user`. Wire up Apollo Client with `setContext` auth link to send the token in every request header.

**Lesson 6 — Authorization: Roles and Ownership**
Authentication answers "who are you?" — authorization answers "what can you do?" Add a `role` field to the User model (`user`, `admin`). Write a `requireAuth(context)` helper to avoid repeating the auth check. Protect mutations by role: only admins can delete; only the owner can update their own record. Implement field-level restrictions: hide sensitive fields like `email` from non-admins in the schema or resolver.

---

### Week 4 — Pagination

**Lesson 7 — Offset Pagination**
Paginate large result sets with `limit` and `offset` arguments — the simplest pattern, closest to SQL `LIMIT`/`SKIP`. Add `limit` and `offset` to a query, implement in the resolver with `.limit()` and `.skip()` in Mongoose, and return a `PaginatedResult` type containing `items`, `total`, and `hasMore`. Build page controls in React using `useQuery` variables.

**Lesson 8 — Cursor-Based Pagination (Relay Style)**
The industry-standard pagination pattern used by GitHub, Shopify, and most large GraphQL APIs. Instead of a page number, use a cursor — an opaque pointer to the last item seen. Return a `Connection` type with `edges` (each containing a `node` and a `cursor`) and `pageInfo` (`hasNextPage`, `endCursor`). Implement `first` and `after` arguments. Understand why cursor pagination handles live data better than offset.

---

### Week 5 — Real-Time with Subscriptions

**Lesson 9 — GraphQL Subscriptions Setup**
Subscriptions push data from server to client over a persistent WebSocket connection. Set up `graphql-ws` alongside Apollo Server 4, define a `Subscription` type in the schema, and use `PubSub` from `graphql-subscriptions` to publish events when mutations fire. Test subscriptions in Apollo Sandbox's subscription tab. Understand the difference between queries/mutations (HTTP) and subscriptions (WebSocket).

**Lesson 10 — Subscriptions in React**
Connect the React client to subscriptions with `useSubscription`. Add a `WebSocketLink` to Apollo Client alongside the `HttpLink`, and use `split` to route subscription operations to the WebSocket and everything else to HTTP. Build a live-updating list that adds new items as they arrive without a page refresh. Handle the loading and error states for subscriptions.

---

### Week 6 — Performance and Advanced Schema

**Lesson 11 — N+1 Problem and DataLoader**
The N+1 problem: fetching a list of 100 items triggers 100 additional queries for each item's related data — 101 queries total. Identify it with `console.log` in a resolver. Fix it with `DataLoader`, a utility that batches and caches requests within a single tick of the event loop. Write a DataLoader for a nested field and measure the reduction in database queries.

**Lesson 12 — Advanced Schema Design**
Four patterns that make schemas more expressive: (1) **Unions** — a field that can return one of several types (search results that return `Post | User | Product`); (2) **Interfaces** — shared fields across types (`Node` interface with `id`, common in Relay); (3) **Custom scalars** — `Date`, `Email`, `URL` with built-in validation; (4) **Input types** — reusable argument objects for mutations (`CreatePostInput`, `UpdatePostInput`). Apply at least two to the final project schema.

---

### Week 7 — Testing and Deployment

**Lesson 13 — Testing GraphQL APIs**
Write tests that give you confidence without mocking too much. Unit test pure resolver functions with plain JavaScript. Integration-test the full Apollo Server with `executeOperation` — no HTTP needed, just pass a query string and check the response. Use `vitest` or `jest`. Test the happy path, error cases, and auth guards. Cover at least: one query, one mutation, one auth-protected operation.

**Lesson 14 (Final Week) — Deployment**
Ship the final project to the internet. Deploy the Apollo Server to [Railway](https://railway.app) or [Render](https://render.com) — both support Node.js with zero-config deploys from GitHub. Deploy the React + Vite client to [Vercel](https://vercel.com) or [Netlify](https://netlify.com). Set environment variables (MongoDB URI, JWT secret) in the platform dashboard — never commit them. Update the Apollo Client URI to the production server URL. Verify the full app works end-to-end in production.

---

## Final Project

The final project extends or rebuilds a GraphQL application using the advanced features from this course. Requirements are in [Assignments/FinalProjectSpec.md](Assignments/FinalProjectSpec.md).

**Minimum requirements for this term:**
- MongoDB persistence (not in-memory)
- JWT authentication with at least one protected route
- Pagination on at least one list query
- Deployed — both server and client live at a public URL

**Stretch goals:**
- Real-time subscription for at least one event
- Role-based authorization (admin vs. user)
- DataLoader for a nested resolver
- Full test suite for server resolvers

---

## Course Repo Setup

Same repo from last term works fine — add a new top-level folder, e.g. `acs-4330-advanced/`, with sub-folders per lesson.

```
acs-4330-advanced/
  lesson-01/
    server.js
    package.json
  lesson-05/
    server.js
    package.json
  final-project/
    server/
    client/
```

Not every lesson produces code to push — check each lesson's "After This Lesson" section.

---

## Resources

- https://www.apollographql.com/docs/apollo-server
- https://www.apollographql.com/docs/react
- https://mongoosejs.com/docs
- https://github.com/graphql/dataloader
- https://relay.dev/graphql/connections.htm (Relay cursor spec)
- https://graphql-ws.netlify.app (WebSocket subscriptions)
- https://jwt.io
