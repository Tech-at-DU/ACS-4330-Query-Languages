# ACS 4330 - Final Project

**Assigned:** Class 9  
**Presentations:** Class 13  
**Final submission:** Class 14

<!-- > -->

## Description

Build a full-stack GraphQL application of your own design. Your project should demonstrate everything you've learned: schema design, resolvers, mutations, nested types, and a React client using Apollo Client.

This is a portfolio piece — pick something you'd actually want to show to a future employer.

<!-- > -->

## Project Ideas

Need inspiration? Here are some starting points:

- **Bookshelf tracker** — books, authors, genres; track what you've read
- **Recipe app** — recipes, ingredients, tags; build a personal cookbook
- **Game tracker** — games, platforms, genres; log what you've played
- **Movie watchlist** — movies, actors, genres; rate and review films
- **Pet adoption board** — pets, shelters, species; available for adoption
- **Study tracker** — topics, resources, sessions; log your learning

Or pitch your own idea — submit it as part of your proposal.

<!-- > -->

## Milestones

| Class | Deliverable |
|-------|------------|
| 9  | **Proposal** — what you're building, your types, your relationships |
| 10 | **In-memory MVP** — server runs, basic queries and mutations work in Apollo Sandbox |
| 11 | **React client** — frontend connected to server, data displays, mutations work from the UI |
| 12 | **Polish** — error handling, loading states, stretch goals if time allows |
| 13 | **Presentation** — 5 min demo + walkthrough of your schema and one resolver |
| 14 | **Final submission** — code pushed to GitHub, link submitted to GradeScope |

<!-- > -->

## Proposal (Class 9)

Submit a short written proposal with:

1. **What your app does** — one sentence
2. **Your types** — list each type and its fields
3. **Your relationships** — which types connect to each other and how
4. **Your queries** — what a user can fetch
5. **Your mutations** — what a user can create, update, or delete

Example proposal:

> **Bookshelf Tracker**
>
> Types: `Book` (title, year, rating), `Author` (name, bio), `Genre` (name)
>
> Relationships: `Book → Author` (each book has an author), `Book → [Genre]` (each book has many genres)
>
> Queries: `books`, `book(id)`, `author(id)`, `authors`
>
> Mutations: `addBook`, `updateBook`, `deleteBook`, `addAuthor`

<!-- > -->

## Requirements

### Server

- [ ] Apollo Server 4 with ESM (`import`/`export`, `"type": "module"` in package.json)
- [ ] At least **3 GraphQL types** (not counting `Query` and `Mutation`)
- [ ] At least **1 relationship** between types — implemented with a nested resolver
- [ ] At least **2 queries** (e.g. get one by ID, get all)
- [ ] At least **2 mutations** (e.g. create + update or delete)
- [ ] At least one query accepts an **argument** (e.g. `book(id: ID!)`)

### Client

- [ ] React + Vite with Apollo Client
- [ ] `ApolloProvider` wraps the app in `main.jsx`
- [ ] At least one query uses **variables** (not hardcoded values)
- [ ] Uses `useLazyQuery` or `useQuery` with variables
- [ ] At least one mutation fires from the UI (form submit, button click, etc.)
- [ ] Loading and error states handled — UI doesn't crash or show blank screen

### Code Quality

- [ ] `.gitignore` includes `node_modules` and `.env`
- [ ] No API keys committed to GitHub
- [ ] Schema makes sense — types and fields are named clearly
- [ ] Resolver structure matches schema (one resolver key per type that has nested fields)

<!-- > -->

## Stretch Goals

These are optional. Prioritize the requirements first.

- **MongoDB** — swap in-memory arrays for a real database (covered in Class 10)
- **Subscriptions** — real-time updates with `useSubscription` (covered in Class 11)
- **External API** — wrap a public REST API in a GraphQL resolver
- **Authentication** — pass a user token via context, restrict mutations
- **Deployed** — server and client live on the internet

<!-- > -->

## Presentation (Class 13)

5 minutes. Show:

1. What your app does — quick demo
2. Your schema — walk through your types and one relationship
3. One resolver — explain what it does and why it's structured that way
4. What was hardest — one thing that took longer than expected

No slides required. Just open your code and your app.

<!-- > -->

## Rubric

| | 1 — Needs Improvement | 2 — Basic | 3 — Proficient | 4 — Advanced |
|:---|:---|:---|:---|:---|
| **Schema design** | Fewer than 3 types, no relationships | 3 types present but relationships missing or incorrect | 3+ types with at least 1 working relationship | Schema is well-designed, relationships are meaningful and match the domain |
| **Resolvers** | Resolvers missing or don't return correct data | Basic Query resolvers work; mutations incomplete | Queries and mutations work; nested resolver returns correct data | Resolvers handle edge cases, null safety, argument validation |
| **React client** | Client doesn't connect to server | Client fetches data but crashes on errors or loading | Queries and mutations work from UI; loading and error states handled | UI is polished; variables used correctly; multiple components |
| **Requirements met** | Fewer than half the requirements | Most requirements met with minor gaps | All requirements met | All requirements met plus at least one stretch goal |
