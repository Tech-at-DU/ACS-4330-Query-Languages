# ACS 4330 - Query Languages

This course covers GraphQL — a query language for APIs that replaces REST with a single flexible endpoint. Instead of multiple endpoints returning fixed data shapes, GraphQL lets clients request exactly the fields they need, nothing more.

GraphQL is used in production at Facebook, GitHub, Shopify, Twitter, Coursera, Airbnb, and hundreds of other companies. It's a marketable skill and a genuinely better way to design APIs.

## Course Specifics

**Course Delivery**: Independent study | 7 weeks | 75 total hours (~10 hours/week)

**Course Credits**: 3 units

### Prerequisites

- Senior Standing
- Instructor Approval
- Familiarity with JavaScript, Node.js, and React

### Learning Objectives

By the end of this course you will be able to:

1. Compare GraphQL with REST and explain the tradeoffs
2. Write GraphQL queries, mutations, and subscriptions
3. Build a GraphQL API with Apollo Server
4. Connect a React frontend to a GraphQL API with Apollo Client
5. Design a schema with multiple related types and nested resolvers
6. Persist data with MongoDB and protect routes with JWT authentication

## Before You Start

### Node.js

This course requires Node.js 18 or higher. Check your version:

```bash
node --version
```

If the output is below `v18`, download the latest LTS release from [nodejs.org](https://nodejs.org) and reinstall.

### Course Repo

Create one GitHub repository for all your coursework — e.g. `acs-4330-graphql`. You'll use this repo for every lesson. See [Course Repo Setup](#course-repo-setup) below for the folder structure.

Always include a `.gitignore` in every project folder. Use [gitignore.io](https://www.toptal.com/developers/gitignore/api/node) to generate one for Node — make sure it excludes `node_modules` and `.env`.

## Schedule

**Course Dates:** August 25 – October 8, 2026

**Format:** Independent study. Work through two lessons per week on your own. Meet with your instructor once per week to review progress and ask questions.

| Week | Lessons | Topics | Graded |
|:-----|:--------|:-------|:-------|
| 1 | [Lesson 1] + [Lesson 2] | GraphQL intro, schemas, types | L1 queries |
| 2 | [Lesson 3] + [Lesson 4] | Apollo Server + React client | L3 server, L4 app |
| 3 | [Lesson 5] + [Lesson 6] | Lab checkpoint + mutations | — |
| 4 | [Lesson 7] + [Lesson 8] | Query variables + nested resolvers | — |
| 5 | [Lesson 9] + start [Final Project] | Context + project proposal | L9 proposal |
| 6 | [Lesson 10] + [Lesson 11] | MongoDB + Apollo cache | — |
| 7 | [Lesson 12] + presentations | Authentication + final project demo | Final project, final assessment |

[Lesson 1]: Lessons/Lesson-1.md
[Lesson 2]: Lessons/Lesson-2.md
[Lesson 3]: Lessons/Lesson-3.md
[Lesson 4]: Lessons/Lesson-4.md
[Lesson 5]: Lessons/Lesson-5.md
[Lesson 6]: Lessons/Lesson-6.md
[Lesson 7]: Lessons/Lesson-7.md
[Lesson 8]: Lessons/Lesson-8.md
[Lesson 9]: Lessons/Lesson-9.md
[Lesson 10]: Lessons/Lesson-graphql-mongo.md
[Lesson 11]: Lessons/Lesson-apollo-cache.md
[Lesson 12]: Lessons/Lesson-12.md
[Final Project]: Assignments/FinalProjectSpec.md


## Course Repo Setup

At the start of the course, create one GitHub repository for all your work — e.g. `acs-4330-graphql`. Then add your name and repo link to the [course tracker spreadsheet] (link shared by your instructor at the first meeting).

Organize your repo like this:

```
acs-4330-graphql/
  lesson-01/
    README.md       ← written answers and query responses
  lesson-02/
    server.js
    package.json
    README.md
  lesson-03/
    ...
  final-project/
    PROPOSAL.md
    server/
    client/
```

Each lesson tells you what to push and where. Written answers go in `README.md` inside the lesson folder. Code goes alongside it.

### Submitting Work

For each lesson, the "After This Lesson" section tells you what to do. Most work falls into one of two categories:

**Mark your tracker** — self-report when done. Your instructor may spot-check.

**Paste a link in the tracker** — paste a direct GitHub link to the specific file or folder. Your instructor will review it.

| Week | What to submit | How |
|------|---------------|-----|
| 1 (L1) | Link to `lesson-01/README.md` — your 10 GraphQL queries | Paste link in tracker |
| 1 (L2) | Apollo Server challenges | Mark tracker |
| 2 (L3) | Link to `lesson-03/` — your GraphQL weather API | Paste link in tracker |
| 2 (L4) | Link to `lesson-04/` — your React weather app | Paste link in tracker |
| 3 (L5) | Lesson 5 self-assessment checklist | Mark tracker |
| 3 (L6) | Mutations server | Mark tracker |
| 4 (L7) | Updated React client with `useLazyQuery` | Mark tracker |
| 4 (L8) | Nested resolvers project | Mark tracker |
| 5 (L9) | Link to `final-project/PROPOSAL.md` | Paste link in tracker |
| 6 (L10) | Add MongoDB to your final project | Mark tracker |
| 6 (L11) | Add Apollo cache handling to your final project | Mark tracker |
| 7 (L12) | Add authentication to your final project | Mark tracker |
| 7 | Final project demo — push `final-project/`, paste repo + deployed URL in tracker | Paste links in tracker |
| 7 | Final assessment — push `final-assessment/README.md`, paste link in tracker | Paste link in tracker |

Link to Course Tracker: https://docs.google.com/spreadsheets/d/1Dbz53bRigu6FwVzyEKY2Gxi4ySKDsEamsvRMLccOCbM

## When You're Stuck

Getting stuck is normal. Here's how to work through it before the next check-in.

**1. Read the error message in full.** The first line usually tells you what went wrong. Copy it exactly.

**2. Check the right place for the error:**
- Terminal → server problem (resolver, schema, Apollo Server startup)
- Browser console → client problem (Apollo Client, React component, network)
- Apollo Sandbox → test the query in isolation to rule out the server

**3. Add a `console.log` just before the failure.** Confirm the value you're passing is what you think it is.

**4. Search the exact error message.** Include the library name: `"Apollo Server 4 context function"`, `"useLazyQuery undefined"`, etc.

**5. Check the official docs:**
- Apollo Server: https://www.apollographql.com/docs/apollo-server
- Apollo Client: https://www.apollographql.com/docs/react
- GraphQL spec: https://graphql.org/learn
- MDN (JavaScript): https://developer.mozilla.org

**6. Still stuck?** Note the exact error message, what you tried, and what you expected to happen. Bring it to the weekly check-in. The more specific your question, the faster it gets answered.

## Independent Study Expectations

This course runs as independent study. You work through the lessons on your own schedule and meet with your instructor once per week.

- Work through lessons between check-ins — aim for the pacing in the homework table above
- Push completed work to your course repo before each meeting
- Come to check-ins with specific questions, not just "I'm stuck" — know which step failed and what you tried
- The course is 75 total hours. That's roughly 10 hours per week over 7 weeks

## Evaluation

To pass this course you must:

- Complete all lesson assignments and push them to your course repo
- Complete and present the final project
- Pass the final assessment ≥75%

## Resources

- https://www.apollographql.com/docs/apollo-server
- https://www.apollographql.com/docs/react
- https://graphql.org/learn
- https://rickandmortyapi.com/documentation
- https://github.com/soggybag/BasicGraphQLExample
