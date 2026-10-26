# ACS 4330 Advanced - Final Project

**Assigned:** Week 1
**Presentation and final submission:** Week 7

<!-- > -->

## Description

Take a GraphQL app from "works on my laptop" to **production-ready**. You'll add a real database, authentication, efficient caching, pagination, and either real-time updates or batched data loading, and then deploy it.

You have two options:

- **Extend your ACS 4330 final project** (recommended). You already have the schema and a client, so each week is focused on the new topic.
- **Start a new project.** Keep it small: 2–3 types with at least one relationship is plenty.

You build the project a piece at a time. Each lesson ends with a **Final Project Milestone** that adds that week's topic.

<!-- > -->

## Project Ideas

- **Bookshelf tracker** — books, authors, reading status; users track their own shelves
- **Recipe box** — recipes and ingredients; users save favorites; live "new recipe" feed
- **Event board** — events and RSVPs; organizers edit their own events; live RSVP count
- **Study group finder** — groups and members; paginated group search
- **Pet adoption board** — the Pets app from the lessons, extended with your own features (adoption requests, favorites, photos)

<!-- > -->

## Weekly Milestones

| Week | Milestone |
|:-----|:----------|
| 1 | `PLAN.md` with your types and relationships; MongoDB models; at least one relationship with nested resolvers |
| 2 | React client; every mutation updates the UI without a refresh; one optimistic mutation |
| 3 | Register/login/`me`; protected mutations; one authorization rule (ownership or role) |
| 4 | One paginated list (offset or cursor) |
| 5 | Server switched to Express at `/graphql`; subscription added (if you chose subscriptions) |
| 6 | DataLoader added (if you chose DataLoader); all features working locally |
| 7 | Deployed; README complete; presentation |

Falling behind on a milestone? Bring it up at your weekly check-in rather than skipping it. Later weeks build on earlier ones.

<!-- > -->

## Requirements

### Server

- [ ] Apollo Server with Express, served at `/graphql`
- [ ] **MongoDB** via Mongoose; at least **2 models** with at least **1 relationship** resolved by nested resolvers
- [ ] Mongoose validation on required fields
- [ ] **Authentication**: `register`, `login`, and `me`; passwords hashed; JWT verified in context
- [ ] Every mutation that changes data requires login
- [ ] At least **one authorization rule** beyond login: ownership (only the creator can edit/delete) **or** a role (admin-only action)
- [ ] **Pagination** on at least one list query, with a capped page size
- [ ] **At least one** of:
  - A **subscription** that fires from a mutation
  - A **DataLoader** for at least one nested resolver

### Client

- [ ] React + Vite with Apollo Client
- [ ] Login, register, and logout; token sent with every request
- [ ] Every mutation updates the UI without a page refresh (refetch, cache write, or eviction)
- [ ] At least one **optimistic** mutation
- [ ] Pagination UI (page controls or Load more)
- [ ] Loading and error states handled; protected actions hidden or disabled when logged out

### Deployment

- [ ] Server deployed (Render, Railway, or similar)
- [ ] Client deployed (Vercel, Netlify, or similar)
- [ ] All secrets in environment variables; `.env` never committed
- [ ] Server URLs in the client come from `VITE_` environment variables

### Documentation

- [ ] `final-project/PLAN.md` — types, relationships, and the decisions you made along the way
- [ ] `final-project/README.md` — live URLs, a test account, completed features, and how to run it locally

<!-- > -->

## Stretch Goals

Optional. Finish the requirements first.

- **Both** a subscription and a DataLoader
- Both ownership **and** role-based rules
- Cursor-based (Relay) pagination with `relayStylePagination`
- A filtered subscription with `withFilter`
- Field-level privacy (e.g. email visible only to its owner)
- Tests for your resolvers (e.g. with `vitest` and Apollo Server's `executeOperation`)

<!-- > -->

## Presentation (Week 7)

10 minutes at your final check-in, using your **deployed** app:

1. **Demo** (3 min) — the main features, live
2. **Schema** (2 min) — your types, one relationship, and one embed-vs-reference decision
3. **Deep-dive** (3 min) — one advanced feature: auth rules, pagination, subscription, or DataLoader. Show the code and explain it
4. **Reflection** (2 min) — what was hardest, and what you'd change

<!-- > -->

## Rubric

| | 1 — Needs Improvement | 2 — Basic | 3 — Proficient | 4 — Advanced |
|:---|:---|:---|:---|:---|
| **Data layer** | In-memory data or no relationships | MongoDB models but relationship missing or broken | 2+ models, working relationship, validation | Indexes and embed-vs-reference choices justified in `PLAN.md` |
| **Auth** | No authentication | Login works but mutations unprotected | Protected mutations plus one authorization rule | Correct error codes; UI reflects permissions; logout resets the cache |
| **Client cache** | UI needs refresh after mutations | Some mutations update the UI | All mutations update the UI; one optimistic | Approach fits each mutation (update vs. add vs. delete); failures handled |
| **Pagination** | None | Paginated, but limit not capped or order unstable | Working pagination with capped limit and stable sort | Cursor pagination with a merged cache, or a well-justified offset choice |
| **Real-time / performance** | Neither feature | Attempted, partly working | Subscription **or** DataLoader working | Both, or one done exceptionally well (filters, measured query counts) |
| **Deployment** | Not deployed | Deployed but major features broken | Fully working in production | CORS locked down; env-based config; thorough README |
| **Presentation** | Not presented | Demo only | Demo, schema, and deep-dive | Clear explanation of trade-offs and lessons learned |

To pass, every row must be at **2** or above, and the project must be deployed.
