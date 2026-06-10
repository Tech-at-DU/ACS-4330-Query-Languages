# ACS 4330 - GraphQL Mutations

**Estimated time:** 3–4 hours

**Prerequisites:** Completed Lesson 2. You have a working Apollo Server with a list of things (pets, songs, etc.) and can query them.

Mutations are queries that make changes — they mutate data.

<!-- > -->

## Learning Objectives

1. Describe mutations
1. Define mutation types in a schema
1. Write mutation queries
1. Write mutation resolvers

<!-- > -->

## Before You Start — Self-Check

Use the Feynman technique solo: write down your answers to these questions in plain language, as if explaining to someone who has never heard of GraphQL. Where you can't explain it clearly, that's the gap to fix before continuing.

1. What is GraphQL?
2. How is it different from REST?
3. What is the GraphQL query language?
4. What is the GraphQL schema language?
5. What is a resolver?

Go back to Lessons 1–2 for anything you can't explain clearly.

<!-- > -->

Example reference project: https://github.com/Tech-at-DU/class-6-nested-queries

## Mutations

<!-- > -->

So far you've been using queries to get things from your GraphQL server. This is like a GET request with a REST 😴 server. 

**Mutations** ⚒️ are used to make changes at your GraphQL server. This is like a POST, PUT, or DELETE request with a REST server.

<!-- > -->

Mutations should probably have a name that describes what they do: 

```python
newUser
createUser
makeUserAccount
addUser
```

<!-- > -->

Define a mutation in your schema with type Mutation: 

```python
# GraphQL Schema Language
type Mutation {
	...
}
```

<small>A mutation starts with `type Mutation`</small>

<!-- > -->

Usually a Mutation will take some parameters and resolve to a type. For example you might supply a username and password and resolve/return a User type. You might provide a url and description and Resolve to a Post type. 

<!-- > -->

Here is an example in code.

```python
# GraphQL Schema Language
type Mutation {
	createUser(name: String!): User!
	createPost(url: String!, description: String!): Link!
}
```

<small>Mutations often return the thing they create, User, or Link in this example.</small>

<!-- > -->

When making a mutation **query** you start with the word "mutation"

```python
# GraphQL Query Language 
mutation {
	createUser(name: "Jo") {
		name
		id
	}
}
```

<!-- > -->

Note! Queries start with the key word Query. But we've been omitting it. 

```python
# Query
query {
	getUsers {
		name
	}
}
```

<!-- > -->

## Mutation Challenges 

<!-- > -->

Using your Apollo Server from Lesson 2 (the list of pets, songs, etc.), add mutations to support full CRUD.

The data source is in-memory — changes only persist while the server is running. That's fine for now; persistence is covered later when we add a database.

<!-- > -->

**Challenge 1 - Serve a list of things**

You have a list of things, Pets were used in the examples.

Write a mutation that adds a new thing. 

It should return the thing that was just created. You'll need to include all of the fields that make the type. 

<!-- > -->

For example if the Pet type looked like this: 

```JS
type Pet {
	name: String!
	species: String!
}
```

The mutation might look like this: 

```python
type Mutation {
	addPet(name: String!, species: String!): Pet!
}
```

<!-- > -->

Add a `Mutation` resolver. In Apollo Server, mutations live under `Mutation:` in your resolvers object — separate from `Query:`:

```js
const resolvers = {
  Query: {
    // ... your existing queries
  },
  Mutation: {
    addPet: (_, { name, species }) => {
      const pet = { name, species }
      petList.push(pet)
      return pet
    }
  }
}
```

Test your work with query. Run your server, open Graphiql in your browser and test your mutation. 

<!-- > -->

```python
mutation {
  addPet(name:"Ginger", species:"Cat") {
    name
  }
}
```

Try test all of your things to see if the new was added to the list. 

<!-- > -->

**Challenge 2 - Update**

We need full CRUD functionality! 

So far you have "Create" and "Read". What about "Update"?

To do this you'll need to make a query that supports all of the field a type has, and a supply something to idententify the record to update.

<!-- > -->

Add a new mutation to your scema. It should include all of the fields but they can be null except the id. It should return the type. 

```python
type Mutation {
	...
	updatePet(id: Int!, name: String, species: String): Pet
} 
```

<small>Figure that a null value is a field that will not be updated.</small>

<!-- > -->

Add a resolver under `Mutation:`. Update only fields that were provided:

```js
const resolvers = {
  Query: { /* ... */ },
  Mutation: {
    // ... addPet,
    updatePet: (_, { id, name, species }) => {
      const pet = petList[id]
      if (pet === undefined) {
        return null
      }
      pet.name = name || pet.name
      pet.species = species || pet.species
      return pet
    }
  }
}
```

<!-- > -->

Test your work with a query. 

- Update an element in your list
- Try changing only one field
- Try updated an id that is out of range 

<!-- > -->

**Challenge 3 - Delete**

Make a mutation to delete an element. It should include an id and return the thing that was deleted. 

Write the mutation in your schema. 

Write a resolver to support the mutation.

<!-- > -->

**Test your work.**

- Write a query that deletes an item from your list
  - You should get the deleted item and be able to display its fields
- Try deleting an id that doesn't exist it should return null

<!-- > -->

**Challenge 4 - Mutation Queries**

Write queries that cover all of the CRUD operations you have implemented. Include these in a read me with your project. You should have a query for: 

1. Creating new item
1. Reading a item from your list
1. Updating an item
1. Deleting an item

<!-- > -->

**Challenge 5 - Self Code Review**

Before moving on, review your own code against this checklist:

- [ ] Schema has `type Mutation { ... }` separate from `type Query { ... }`
- [ ] Resolvers object has both `Query: { ... }` and `Mutation: { ... }` keys
- [ ] Each mutation resolver takes `(_, { ...args })` — not just `({ ...args })`
- [ ] `addPet` returns the created pet
- [ ] `updatePet` returns null for an out-of-range id
- [ ] `deletePet` returns the deleted pet (or null if not found)
- [ ] All three mutations tested in Apollo Sandbox with valid and invalid inputs

<!-- > -->

**Challenge 6 - Book List**

Add a new list to your example. This will be a list of books. Each book should have the following fields: 

- title
- author
- isbn

Add the following types to your schema: 

- type Book - defines a book
- books - a query that returns an array of Book
- addBook - a mutation that creates a new book and returns that book
- editBook - a mutation that updates an existing book and returns that book
- deleteBook - a mutation that deletes an existing book

<!-- > -->

## After This Lesson

Complete the challenges above and submit to GradeScope.

<!-- > -->

### Evaluate your work

<!-- > -->

| - | Does not meet expectations | Meets Expectations | Exceeds Expectations |
|:---:|:---:|:---|:---:|
| Comprehension | Can't describe GraphQL mutations | Can describe GraphQL mutations | Could describe potential use cases for GraphQL mutations beyond the examples provided |
| Mutation Queries | Can't write a mutation query | Can write mutation queries | Can write mutation queries that expand on the challenge solutions |
| Mutation Resolvers | Can't write a Mutation resolver | Can write a mutation resolver | Could write mutation resolvers that expand upon the solutions to the challenges |

<!-- > -->

## Resources

- https://odyssey.apollographql.com
