# ACS 4330 - Advanced GraphQL

This course picks up where ACS 4330 Query Languages left off. You've built a GraphQL API with Apollo Server and a React + Apollo Client frontend. This course covers what it takes to make a GraphQL app production-ready: a real database, authentication, cache management, pagination, real-time data, performance, and deployment.

**Course Delivery**: Independent study | 7 weeks | 75 total hours (~10 hours/week)

**Course Credits**: 3 units

## Prerequisites

- Completed ACS 4330 Query Languages
- Comfortable with: GraphQL schemas, resolvers, nested resolvers, mutations, Apollo Server, React + Apollo Client, `useQuery`, `useMutation`
- Node.js 20+, async/await, ES modules (`import`/`export`)

> **Did ACS 4330 Lessons 10–12 (MongoDB, Apollo cache, authentication)?** Weeks 1–3 start with that material, so you can skim it. Each of those weeks then adds a new part: data modeling, optimistic UI, and authorization.

## Learning Objectives

By the end of this course you will be able to:

1. Persist GraphQL data with MongoDB and Mongoose, and model relationships between collections
2. Keep the Apollo Client cache in sync after mutations, including optimistic UI
3. Implement JWT authentication and role- and ownership-based authorization
4. Paginate results with offset and cursor-based (Relay-style) patterns
5. Build real-time features with GraphQL subscriptions
6. Solve the N+1 problem with DataLoader
7. Deploy a full-stack GraphQL app to the internet

---

## Schedule

**Format:** Independent study. Work through one lesson per week on your own, and apply it to your final project. Meet with your instructor once per week.

| Week | Lesson | Topics | Final Project Milestone |
|:-----|:-------|:-------|:------------------------|
| 1 | [Lesson 1] | MongoDB + Mongoose, data modeling | `PLAN.md`; models + one relationship |
| 2 | [Lesson 2] | Apollo cache, optimistic UI | Client updates without refresh |
| 3 | [Lesson 3] | JWT authentication, authorization | Login + one authorization rule |
| 4 | [Lesson 4] | Offset and cursor pagination | One paginated list |
| 5 | [Lesson 5] | Subscriptions (server + React) | Express server; subscription *or*… |
| 6 | [Lesson 6] | N+1 problem, DataLoader | …DataLoader; feature-complete |
| 7 | [Lesson 7] | Deployment, final presentation | Deployed + presented |

[Lesson 1]: Lessons/Lesson-1.md
[Lesson 2]: Lessons/Lesson-2.md
[Lesson 3]: Lessons/Lesson-3.md
[Lesson 4]: Lessons/Lesson-4.md
[Lesson 5]: Lessons/Lesson-5.md
[Lesson 6]: Lessons/Lesson-6.md
[Lesson 7]: Lessons/Lesson-7.md

---

## How Each Week Works

Every lesson has the same structure:

1. **Learning Outcomes and Review** — what you'll learn, and a few questions to answer from memory first
2. **Lesson** — explanations with all the code you need, built on a running **Pets** example app
3. **Challenges** — hands-on practice in the Pets app, ending with a stretch challenge
4. **Final Project Milestone** — apply the week's topic to your own project
5. **After This Lesson** — what to push and mark in the tracker
6. **Evaluate Your Work** — a rubric to check yourself before your check-in

Budget about **4–5 hours for the lesson and challenges** and **2–3 hours for your project** each week. Week 6 is shorter, so you have time to finish your project before deploying.

---

## Lesson Outline

**Week 1 — MongoDB and Data Modeling**
Connect Apollo Server to MongoDB Atlas with Mongoose. Replace in-memory arrays with models and async resolvers, and pass models through context. Then model a relationship: compare embedding with referencing, add a `Shelter` model referenced from `Pet`, resolve it with nested resolvers, and add validation and indexes.

**Week 2 — Apollo Client Cache and Optimistic UI**
See why the UI goes stale after a mutation, and fix it four ways: `refetchQueries`, automatic updates by `id`, `cache.modify` + `writeFragment`, and `cache.evict`. Then make mutations feel instant with `optimisticResponse`: style pending items, and recover when the server fails.

**Week 3 — Authentication and Authorization**
Build `register`, `login`, and `me` with bcrypt and JWT. Verify tokens in context and protect resolvers with a `requireAuth` helper and `GraphQLError` codes. Add roles (admin-only actions) and ownership (only the creator can edit or delete). Send the token from Apollo Client with `SetContextLink`, and reset the cache on login and logout.

**Week 4 — Pagination**
Build offset pagination (`limit`/`offset`, `total`, `hasMore`) with page controls, and see why inserts break it. Then build Relay-style cursor pagination (`edges`, `node`, `cursor`, `pageInfo`) with opaque cursors. Add "Load more" using `fetchMore` and `relayStylePagination`.

**Week 5 — Subscriptions**
Move the server to Express so it can serve HTTP and WebSockets together. Publish events from mutations with `PubSub`, and test subscriptions in Sandbox. In React, route subscriptions with `ApolloLink.split`, show live events with `useSubscription`, and write them into the cache.

**Week 6 — The N+1 Problem and DataLoader**
Measure the N+1 problem with Mongoose debug logging. Fix it with per-request DataLoaders for one-to-one and one-to-many relationships, then measure again. Shorter lesson: use the extra time to finish your project.

**Week 7 — Deployment and Final Presentation**
Prepare the server and client for production (`PORT`, CORS, `VITE_` env vars, Atlas network access). Deploy the server to Render and the client to Vercel, smoke-test everything, and present your project.

---

## Final Project

Take a GraphQL app — ideally your ACS 4330 final project — and make it production-ready, one milestone per week. Full requirements and rubric: [Assignments/FinalProjectSpec.md](Assignments/FinalProjectSpec.md).

**Required:**
- MongoDB persistence with at least one relationship
- JWT authentication plus one authorization rule (ownership or role)
- Every mutation updates the UI without a refresh; at least one optimistic mutation
- Pagination on at least one list
- A subscription **or** a DataLoader
- Deployed — server and client live at public URLs

**Stretch goals:** both a subscription and a DataLoader, cursor pagination, field-level privacy, resolver tests.

---

## Course Repo Setup

Create one GitHub repository for this course — e.g. `acs-4330-advanced`. Add your name and repo link to the course tracker (your instructor will share the link at your first meeting).

```
acs-4330-advanced/
  pets/              ← follow-along app, grows each week
    server/
    client/
  final-project/
    PLAN.md
    README.md
    server/
    client/
```

Include a `.gitignore` in every project folder that excludes `node_modules` and `.env`.

### Submitting Work

| Week | What to submit | How |
|------|---------------|-----|
| 1 | `pets/server/`, `final-project/PLAN.md`, project models | Mark tracker |
| 2 | `pets/client/`, project client cache updates | Mark tracker |
| 3 | Pets auth, project auth | Mark tracker |
| 4 | Pets pagination, project pagination | Mark tracker |
| 5 | Pets subscriptions, project server on Express | Mark tracker |
| 6 | Pets DataLoader, project feature-complete | Mark tracker |
| 7 | Deployed project + `final-project/README.md` | Paste repo link and live URL in tracker |
| 7 | Final presentation | At final check-in |

Your instructor will look at your repo before each check-in, so push before you meet.

---

## When You're Stuck

**1. Read the whole error message.** The first line usually says what went wrong.

**2. Figure out where the error is coming from:**
- Terminal → server (resolver, schema, database connection)
- Browser console → client (Apollo Client, React)
- Apollo Sandbox → test the operation by itself to rule out the client
- Apollo DevTools → see what's in the cache

**3. Add a `console.log` just before the failure** to confirm values are what you expect. For database issues, `mongoose.set('debug', true)` logs every query.

**4. Check the library version.** Many online examples are for older versions. These lessons use Apollo Server 5, Apollo Client 4, graphql-ws 6, graphql-subscriptions 3, and Mongoose 9. If an import path doesn't work, the example is probably outdated.

**5. Search the exact error message**, along with the library name.

**6. Still stuck?** Write down the exact error, what you tried, and what you expected. Bring it to your check-in.

## Independent Study Expectations

- Work through one lesson per week. Lessons build on each other, so don't skip ahead
- Push your work before each check-in
- Come to check-ins with specific questions and a working (or specifically broken) demo
- 75 hours total: about 10 hours per week for 7 weeks

## Evaluation

To pass this course you must:

- Complete all seven lessons and push the Pets app to your course repo
- Complete, deploy, and present the final project, scoring at least **2** in every rubric row

## Resources

- https://www.apollographql.com/docs/apollo-server
- https://www.apollographql.com/docs/react
- https://mongoosejs.com/docs
- https://relay.dev/graphql/connections.htm (Relay cursor spec)
- https://the-guild.dev/graphql/ws (`graphql-ws`)
- https://github.com/graphql/dataloader
- https://jwt.io
