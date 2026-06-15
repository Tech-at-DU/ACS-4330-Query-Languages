# ACS 4330 - Context and Schema Design

**Estimated time:** 3–4 hours

**Prerequisites:** Completed Lessons 6 and 8. You can write resolvers, mutations, and nested resolvers.

<!-- > -->

## Learning Outcomes

By the end of this lesson, you should be able to:

1. Explain what GraphQL context is and why it exists
2. Pass data through context to all resolvers
3. Design a schema for a real-world app with multiple related types
4. Write a project proposal with types, relationships, queries, and mutations

<!-- > -->

## Review

Write these out from memory before continuing:

- What are the four arguments a resolver receives?
- What does the `parent` argument contain in a nested resolver?
- What is the difference between `Query` and `Mutation` in a schema?

<!-- > -->

## 📦 The Resolver Signature

Every resolver has the same four arguments:

```js
const resolvers = {
  Query: {
    myField: (parent, args, context, info) => { ... }
  }
}
```

| Argument | Contains |
|---|---|
| `parent` | Return value of the parent resolver (used for nested types) |
| `args` | Arguments passed to the field in the query |
| `context` | Shared object available to every resolver in the request |
| `info` | Schema metadata — rarely needed |

You've used `parent` and `args`. This lesson covers `context`.

<!-- > -->

## 🌐 What is Context?

Context is a single object, created once per request, passed to every resolver automatically.

Without context, if you needed to pass a database connection or a user identity to a resolver, you'd have to thread it through args — which would pollute your schema. Context solves this.

**Common things passed via context:**

- Database connection or ORM models
- Authenticated user (from a JWT token)
- Request metadata (IP, timestamp)
- Feature flags

<!-- > -->

## Setting Up Context in Apollo Server 4

Pass a `context` function to `startStandaloneServer`. It receives the raw request and returns an object:

```js
const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 },
  context: async ({ req }) => {
    return {
      requestTime: new Date().toISOString(),
    }
  }
})
```

Every resolver can now read `context.requestTime`.

<!-- > -->

Access it in a resolver:

```js
const resolvers = {
  Query: {
    getWeather: async (_, { zip, units = 'imperial' }, context) => {
      console.log('Request at:', context.requestTime)
      // ... fetch weather
    }
  }
}
```

<!-- > -->

## Challenge 1 — Add Context to Your Server

1. Open your Apollo Server (weather or pet server)
2. Add a `context` function that returns `{ requestTime: new Date().toISOString() }`
3. Log `context.requestTime` inside any resolver
4. Run a query in Apollo Sandbox and verify the timestamp appears in the terminal

<!-- > -->

## Challenge 2 — Pass a Value Through Context

Add a custom header value to context.

In the `context` function:

```js
context: async ({ req }) => {
  const clientName = req.headers['x-client-name'] || 'unknown'
  return {
    requestTime: new Date().toISOString(),
    clientName,
  }
}
```

In Apollo Sandbox, open **Headers** and add:

```json
{
  "x-client-name": "my-test-client"
}
```

Log `context.clientName` in a resolver and verify it arrives correctly.

This is the same mechanism used to pass auth tokens — covered in Lesson 12.

<!-- > -->

## 🗂️ Schema Design for a Real Project

Designing a schema for a real app is different from the small examples in earlier lessons. Here are the principles that prevent common mistakes.

<!-- > -->

### Think in types, not endpoints

REST thinks in URLs (`/users`, `/posts`). GraphQL thinks in **types**.

Start by listing the things in your app:

> A bookshelf app has: Books, Authors, Genres, Users

Each becomes a type. Fields on each type are properties of that thing.

<!-- > -->

### Use types when a field has sub-fields

If a field has sub-fields, it's a type — not a scalar.

```graphql
# Wrong — lumps all author data into scalars on Book
type Book {
  title: String!
  authorName: String!       # flat
  authorBio: String!        # flat
}

# Right — Author is its own type
type Book {
  title: String!
  author: Author!           # nested type
}

type Author {
  name: String!
  bio: String
}
```

<!-- > -->

### Connect types with IDs in your data, not embedded objects

Your in-memory data stores IDs. Resolvers do the joining. This mirrors how databases work.

```js
// Data
const books = [
  { id: '1', title: 'Dune', authorId: '1' }
]
const authors = [
  { id: '1', name: 'Frank Herbert' }
]

// Nested resolver joins them
const resolvers = {
  Book: {
    author: (parent) => authors.find(a => a.id === parent.authorId)
  }
}
```

<!-- > -->

### Nullable vs non-null

Use `!` (non-null) when the field will **always** exist. Leave nullable when it might not.

```graphql
type Book {
  id: ID!           # always exists
  title: String!    # always exists
  rating: Float     # might not be rated yet — nullable
  author: Author!   # always has an author
  coAuthor: Author  # might not have one — nullable
}
```

<!-- > -->

### Name mutations clearly

```graphql
type Mutation {
  addBook(title: String!, authorId: ID!): Book!
  updateBook(id: ID!, title: String, rating: Float): Book
  deleteBook(id: ID!): Boolean!
}
```

Pattern: `add`, `update`, `delete` + the type name. Consistent naming makes the API predictable.

<!-- > -->

### Common mistakes

| Mistake | Fix |
|---|---|
| Everything in one giant type | Split into multiple types with relationships |
| Returning raw REST API shapes | Design your GraphQL shape, pick the fields you need |
| Making every field non-null (`!`) | Only use `!` when guaranteed to exist |
| Mutation returns `String` | Return the full updated type so the client can use the result |
| Query returns only one thing | Add both `item(id)` and `items` queries |

<!-- > -->

## Challenge 3 — Design Your Project Schema

Draft the schema for your final project.

1. List every type your app needs
2. Write out the fields for each type — include the GraphQL type for each field
3. Identify relationships: which types connect to which?
4. Write the `Query` type — at minimum `getAll` and `getOne(id)` for your main type
5. Write the `Mutation` type — at minimum `add` and either `update` or `delete`

Write it in GraphQL Schema Language (the `typeDefs` string format).

<!-- > -->

## 📋 Project Proposal

Your proposal is due this class. Submit it to GradeScope.

**Required sections:**

**1. What your app does** (one sentence)

> A bookshelf tracker that lets users log books they've read, rate them, and browse by author or genre.

**2. Types and fields**

```graphql
type Book {
  id: ID!
  title: String!
  rating: Float
  author: Author!
  genres: [Genre!]!
}

type Author {
  id: ID!
  name: String!
  books: [Book!]!
}

type Genre {
  id: ID!
  name: String!
}
```

**3. Relationships**

- `Book` → `Author` (each book has one author)
- `Author` → `[Book]` (each author has many books)
- `Book` → `[Genre]` (each book has many genres)

**4. Queries**

```graphql
type Query {
  books: [Book!]!
  book(id: ID!): Book
  authors: [Author!]!
  author(id: ID!): Author
}
```

**5. Mutations**

```graphql
type Mutation {
  addBook(title: String!, authorId: ID!): Book!
  updateBook(id: ID!, rating: Float): Book
  deleteBook(id: ID!): Boolean!
}
```

<!-- > -->

## After This Lesson

- Submit your project proposal to GradeScope
- Begin building your server with in-memory data arrays
- Schema should match your proposal — you will migrate to MongoDB in Lesson 10

<!-- > -->

### Evaluate Your Work

| | Does not meet expectations | Meets expectations | Exceeds expectations |
|:---:|:---:|:---:|:---:|
| Context | Can't explain what context is | Passes data through context and accesses it in a resolver | Uses context to pass multiple values; explains when it's needed |
| Schema design | Schema has fewer than 3 types or no relationships | 3+ types with at least 1 relationship; queries and mutations defined | Schema is well-named, relationships make sense for the domain, nullable fields are used correctly |
| Proposal | Missing sections | All 5 sections present | Types, relationships, queries, and mutations are complete and consistent |

<!-- > -->

## Resources

- https://www.apollographql.com/docs/apollo-server/data/context
- https://graphql.org/learn/schema
