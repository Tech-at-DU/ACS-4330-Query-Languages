# 4330 Query Languages

ACS 4330 Query Languages explores modern alternatives to REST. In this class you will learn to use languages that facilitate communication between client and server applications, expand and improve on the tired and frustrating REST paradigm. 

Query languages solve many of the problems of REST. Modern query languages such as GraphQL were made with the goal of solving all of the problems of REST. 

Using Modern query languages, you will be able to define requests at your client and receive only the data you requested. Imagine making multiple nested requests with a single query and only receiving the data you asked for. Query languages are the future of web APIs and are currently used by hundreds of companies like: FaceBook/Meta, Twitter, GitHub, AirBnB, Coursera, Intuit, Lyft, Paypal, Pinterest, and more. 

## GraphQL

Learn GraphQL the better replacement for REST! Invented at Facebook to solve problems imposed by REST. GraphQL is an open source alternative that offers a new way of managing network resources. 

### Why you should know this (optional)

GraphQL provides many advantages over REST. It's used by all of the biggest services. 

FaceBook, Coursera, GitHub and many others 

GraphQL is built with a schema and strong types. GraphQL provides reliablity through it's strong typing system. GraphQL provides a solution to over fetching and unders fetching data, and allows front end requests to determine what data is returned from an endpoint. 

If you want to work with a future of network resources learn GraphQL. 

## Course Specifics

**Course Delivery**: online | 7 weeks | 14 sessions

**Course Credits**: 3 units | 37.5 Seat Hours | 75 Total Hours

### Prerequisites  

- Senior Standing
- Instructor Approval

### Learning Objectives

Students by the end of the course will be able to ...

1. Compare GraphQL with REST
1. Describe the Features of the GraphQL language
1. Write a basic GraphQL Query and Schema
1. Implement GraphQL in a CRUD Application

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

**Course Dates:** August 24 – October 9, 2026

**Class Times:** Tuesday and Thursday at 4:00 PM - 6:45 PM

| Class |    Date   |                Topics                  | Assignment |
|:-----|:---------:|:--------------------------------------:|:-----------|
|  1  | Tue, Aug 25 | [Lesson 1 - GraphQL Intro]             | Query Challenges |
|  2  | Thu, Aug 27 | [Lesson 2 - GraphQL Schemas and Types] | GraphQL Resolver Challenges |
|  3  | Tue, Sep  1 | [Lesson 3 - GraphQL + Apollo]          | GraphQL OpenWeatherMap API |
|  4  | Thu, Sep  3 | [Lesson 4 - React Intro]               | React GraphQL Weather |
|  5  | Tue, Sep  8 | [Lesson 5 - React and GraphQL]         | - |
|  6  | Thu, Sep 10 | [Lesson 6 - GraphQL Mutations]         | - |
|  7  | Tue, Sep 15 | [Lesson 7 - Query Variables]                      | - |
|  8  | Thu, Sep 17 | [Nested resolvers]                     | - |
|  9  | Tue, Sep 22 | [Lesson 9 - Context]                   | [Final Project] proposal due |
|  10 | Thu, Sep 24 | [Lesson 10 - MongoDB]                  | - |
|  11 | Tue, Sep 29 | [Lesson 11 - Apollo Cache]             | - |
|  12 | Thu, Oct  1 | [Lesson 12 - Authentication]           | - |
|  13 | Tue, Oct  6 | Final Presentations                    | - |
|  14 | Thu, Oct  8 | Final Assessment                       | - |

[Lesson 1 - GraphQL Intro]: Lessons/Lesson-1.md
[Lesson 2 - GraphQL Schemas and Types]: Lessons/Lesson-2.md
[Lesson 3 - GraphQL + Apollo]: Lessons/Lesson-3.md
[Lesson 4 - React Intro]: Lessons/Lesson-4.md
[Lesson 5 - React and GraphQL]: Lessons/Lesson-5.md
[Lesson 6 - GraphQL Mutations]: Lessons/Lesson-6.md
[Lesson 7 - Query Variables]: Lessons/Lesson-7.md
[Lesson 8 - Nested Resolvers]: Lessons/Lesson-8.md
[Lesson 9 - Context]: Lessons/Lesson-9.md
[Lesson 10 - MongoDB]: Lessons/Lesson-graphql-mongo.md
[Lesson 11 - Apollo Cache]: Lessons/Lesson-apollo-cache.md
[Lesson 12 - Authentication]: Lessons/Lesson-12.md

[Nested resolvers]: Lessons/Lesson-8.md

[Final Project]: Assignments/FinalProjectSpec.md

## Course Repo Setup

At the start of the course, create one GitHub repository for all your work: e.g. `acs-4330-graphql`. Share the link with your instructor.

Organize it like this:

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

### Homework

Complete the work for each lesson before your next weekly check-in. Bring questions — stuck spots are better discussed in person than via messages.

| Class | Work due by next check-in |
|-------|--------------------------|
| 1  | Watch howtographql.com intro videos. Add query answers to `lesson-01/README.md` |
| 2  | Complete Lesson 2 challenges. Push server to `lesson-02/` |
| 3  | Complete Lesson 3 challenges. Push weather API to `lesson-03/` |
| 4  | Complete Lesson 4 challenges. Push React weather app to `lesson-04/` |
| 5  | Finish any open items from Lessons 3–4. Pass all checklist items in Lesson 5 |
| 6  | Complete Lesson 6 challenges. Push mutations server to `lesson-06/` |
| 7  | Complete Lesson 7 challenges. Push updated React client to `lesson-07/` |
| 8  | Complete Lesson 8 challenges. Push nested resolvers project to `lesson-08/` |
| 9  | Write project proposal. Push to `final-project/PROPOSAL.md` |
| 10 | Add MongoDB to your project server. Push to `lesson-10/` |
| 11 | Add cache handling to React client. Push to `lesson-11/` |
| 12 | Add authentication. Push to `lesson-12/` |
| 13 | Final project presentations — demo ready |
| 14 | Final assessment. Push answers to `final-assessment/README.md` |

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
- The course is 75 total hours. That's roughly 5–6 hours per week over 13 weeks

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
