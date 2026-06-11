# ACS 4330 - GraphQL Intro

**Estimated time:** 2–3 hours

**Prerequisites:** Basic JavaScript and HTTP concepts (you know what a GET request is).

<!-- > -->

# Why you should know this?

GraphQL represents a new way to work with network transactions. It provides many benefits over REST.

<!-- > -->

## Learning Objectives

1. Compare REST with GraphQL
1. Define RESTful routes
1. Describe the benefits of GraphQL
1. Compare and contrast REST and GraphQL
1. Write GraphQL Queries

<!-- > -->

## Before You Start

ELI5 (Explain Like I'm 5). Write a one-sentence answer to one of these in your own words before reading further:

- How do web pages work?
- How do web browsers work?
- What are web APIs?

<!-- > -->
## What makes the web work? 

These are two different approaches to online data transmission. These describe how data is passed around over the internet.

REST 😴 and SOAP 🧼

<!-- > -->

### SOAP 🧼

SOAP (Simple Object Access Protocol) is the **official** standard maintained by the W3C. 

<!-- > -->

SOAP uses XML exclusively! XML is a format for storing and transmitting data. Below a block of simple XML:

```xml
<book>
  <isbn>1929394</isbn>
  <author>Secret Agent Man</author>
  ...
</book>
```

<small>xml: eXtensible Markup Language</small>

<!-- > -->

### REST 😴

REST (Representational State Transfer) is an **architectural 🏛 principal** and guideline, used to create public APIs.

<small>Wikipedia says: REST is a software architectural *style*...</small>

<!-- > -->

## How does REST 😴 work?

A REST API is an API that follows REST-ful Routing. REST-ful routing is a set of conventions for implementing **CRUD** applications on an HTTP server. 

<!-- > -->

These conventions are common rules around the type of **HTTP request** and the **URLS** that are used to Create, Read, Update, or Delete data on a server.

<!-- > -->

Fill in the CRUD operation for each route in the table below:

| URL | HTTP Method/Type | CRUD Operation |
| ----------- | ----------- |----------- |
| /posts | POST | ??? <!--Create new post-->|
| /posts| GET | ??? <!--Read all posts--> |
| /posts/10 | PUT | ??? <!--Update post 10--> |
| /posts/23 | DELETE | ??? <!--Delete post 23--> |
| /posts/5 | GET | ??? <!--Read post 5--> |
| /users/2/posts | POST | ??? <!--Create a post for User 2--> |

<small>What would be the operations for each of these routes? What do they do?</small>

<!-- > -->

Here's a real API.

The Rick and Morty API uses the following routes:

- https://rickandmortyapi.com/api/character/`<id\>`
- https://rickandmortyapi.com/api/location/`<id\>`
- https://rickandmortyapi.com/api/episode/`<id\>`

<small>Notice: There is one endpoint for each resource type.</small>

<!-- > -->

## What's the difference between REST and GraphQL?

<!-- > -->

Unlike REST 😴 a GraphQL 😎 server would use a single ☝️ endpoint to serve all of its resources. 

<small>The Rick and Morty REST API had 3 endpoints!</small> 

<!-- > -->

**Compare REST and GraphQL**

- REST 😴 - Multiple endpoints 🖐
- GrapQL 😎 - Single endpoint ☝️

<!-- > -->

### Try out REST 😴 with the Rick and Morty API.

<!-- > -->

Try the character endpoint. Open these URLs in your browser:

- https://rickandmortyapi.com/api/character/1 - Rick Sanchez
- https://rickandmortyapi.com/api/character/2 - Morty Smith
- https://rickandmortyapi.com/api/character/3 - Summer Smith
- https://rickandmortyapi.com/api/character/4 - Beth Smith

**Challenge:** find Jerry Smith, Birdperson, and Mr. Meeseeks

<!-- > -->

Try the location endpoint.

- https://rickandmortyapi.com/api/location/1 - Earth (C-137)
- https://rickandmortyapi.com/api/location/2 - Abadango
- https://rickandmortyapi.com/api/location/3 - Citadel of Ricks

**Challenge:** find the Anatomy Park location and the Immortality Field Resort

<!-- > -->

## GraphQL 😎

<!-- > -->

With GraphQL 😎 you will send a query that might look like this: 

```JS
{
  posts {
    title
  }
}
```

Or this: 

```JS
{
  users {
    name
  }
}
```

<!-- > -->

A query begins with brackets `{}`: 

```JS
{
  ...
}
```

<!-- > -->

Next, add a type and any parameters for that type.

In this case, `character` is our type and `id` is the parameter

```js
{
  character(id: 1) {
    ...
  }
}
```

<!-- > -->

Add fields that you want. Note that `character()` returns a Character type and we can only include fields that exist on Character!

```js
{
  character(id: 1) {
    name
    species
    ...
  }
}
```

<!-- > -->

### Try out GraphQL 😎 with the Rick and Morty API!

To do this you'll use the built-in GraphiQL explorer. It's a web page that lets you write GraphQL queries and see the results.

> GraphiQL is a GraphQL graphical explorer. Write a query in the left pane, click run, view results on the right.

<!-- > -->

Open the GraphiQL browser:

https://rickandmortyapi.com/graphql

- Type a query on the left side
- Click the ▶️ button at the top
- Look 👁 at the results on the right
- Try the following queries...

<!-- > -->

### Challenge: Get characters with GraphQL 😎

```graphql
# Rick Sanchez
{
  character(id: 1) {
    name
  }
}
```

<small>Challenge: change the id to find Morty (2), Summer (3), Beth (4), and Jerry (5)</small>

<!-- > -->

# Why use GraphQL?

## Over Fetching

Over fetching occurs when you make a request and receive more 🗑 data than you need. 

This happens often. Think of all of those fields that you never use. 🤔 

<!-- > -->

Look at the results that are returned with <br> the REST response vs the GraphQL 😎 response. 

**What's the difference? 🤔**

<!-- > -->

The REST API returns the following when you request `https://rickandmortyapi.com/api/character/1`:

```JS
{
  "id": 1,
  "name": "Rick Sanchez",
  "status": "Alive",
  "species": "Human",
  "type": "",
  "gender": "Male",
  "origin": {
    "name": "Earth (C-137)",
    "url": "https://rickandmortyapi.com/api/location/1"
  },
  "location": {
    "name": "Citadel of Ricks",
    "url": "https://rickandmortyapi.com/api/location/3"
  },
  "image": "https://rickandmortyapi.com/api/character/avatar/1.jpeg",
  "episode": [
    "https://rickandmortyapi.com/api/episode/1",
    "https://rickandmortyapi.com/api/episode/2",
    "... 49 more episode URLs"
  ],
  "url": "https://rickandmortyapi.com/api/character/1",
  "created": "2017-11-04T18:48:46.250Z"
}
```

<!-- > -->

With GraphQL 😎 :

```graphql
# We used this:
{
  character(id: 1) {
    name
  }
}

# to get:
{
  "data": {
    "character": {
      "name": "Rick Sanchez"
    }
  }
}
```

<small>If we needed more we could ask for more!</small>

<!-- > -->

If we *only* wanted the `name` field the GraphQL 😎 query would have saved a lot of bandwidth! 🗜

<!-- > -->

Describe the fields you want in the query:

```graphql
{
  character(id: 2) {
    name
    species  # includes species
  }
}
```

<small>Challenge: Get Morty's name and status. Get Summer's name and gender.</small>

<!-- > -->

**Compare REST with GraphQL**

- GraphQL 😎 - we describe what we want <br> and the server returns 🎁 data that matches the query. 
- REST 😴 - you often get more than you need (**over fetching**)

<!-- > -->

## Under Fetching 🥚

Under fetching 🥚 occurs when you don't get all of the data you need in a single request and have to make another request to get the data you require. 

<!-- > -->

**Challenge:** Use REST to find Rick's origin location name. 🌍

- Step 1: https://rickandmortyapi.com/api/character/1 — find the `origin.url`
- Step 2: open that URL to get the location details

<!-- > -->

### Challenge:

Use the REST API to find the origin location of:

1. Morty Smith (id: 2)
1. Summer Smith (id: 3)
1. Beth Smith (id: 4)

https://rickandmortyapi.com/api/character/2

<!-- > -->

**What happened? 🧐**

Each time you found a character, *you had to make a second request* to find their origin location. <br>
<small>(under fetching 🥚)</small>

Along the way, you loaded *more* data than you needed <small>(over fetching 🗑)</small>.

<!-- > -->

**Try it with GraphQL 😎**

```graphql
{
  character(id: 1) {
    name
    origin {
      name
    }
  }
}
```

<small>In a single query we get the character's name and their origin location — no second request needed.</small>

<!-- > -->

### Challenge:

1. Get Morty's name and origin location name
1. Get Summer's name and current location name
1. Get Rick's name, species, and origin location name
1. Get the dimension of Rick's origin location

<!-- > -->

**Compare REST with GraphQL**

- REST 😴 - over fetches, under fetches, or does both!
- GraphQL 😎 - fetches only what you ask for with a single ☝️ query!

<!-- > -->

## GraphQL vs REST

**Reflect:** Before reading the comparison below, write down as many pros and cons of REST and GraphQL as you can from memory. Then compare with the list below — what did you miss?

<!-- > -->

**REST** 😴

- Requires multiple endpoints. Makes for a complex API.
- Harder to make changes to your API. 
- Often over fetches providing more data than you, which eating bandwidth
- Often under fetches, requiring more complex queries and more bandwidth. 

<!-- > -->

**GraphQL** 😎
 
- Uses a single endpoint.
- Easier to manage
- More tolerant to changes
- Fetches only what you ask for
- Prevents over and under fetching 
- Queries return what you ask for
- Saves bandwidth

<!-- > -->

**GraphQL** 😎 provides other benefits!
 
- Type safety 🛡
  - GrapgQL is type safe. Every data element has an associated type. You can see this in the interface!
  - Types are defined in the schema
- Introspection 🔎
  - Since GraphQL is backed by a schema it can predict the fields that are available and the type for each field. You can see this in the UI as it provides code hints.
  - You can browser the structure of a GraphQL server via the docs (look for the docs tab)

<!-- > -->

## Core features of GraphQL

GraphQL is a query language and a spec for a serverside application. 

<!-- > -->

- Query Language
  - Query
  - Mutation
  - Subscription
- Schema Definition Language
  - Strong Typing
  - Introspection

<!-- > -->

**So, What is GraphQL** 🤔

<!-- > -->

> GraphQL is an open-source data query and manipulation language for APIs, and a runtime for fulfilling queries with existing data.

<small>From wikipedia</Small>

<!-- > -->

### Aliases 

<!-- > -->

Since the query describes the structure of what is returned sometimes you need to change the names. 

<!-- > -->

Consider a scenario where you need two characters:

```graphql
{
  character(id: 1) {
    name
  }

  character(id: 2) {
    name
  }
}
```

<small>(Doesn't work!)</small>

<!-- > -->

The results would have a problem:

```JS
{
  "data": {
    "character": { // <-- Duplicate field!
      "name": "Rick Sanchez"
    },
    "character": { // <-- Duplicate field!
      "name": "Morty Smith"
    }
  }
}
```

<small>(Hypothetical results from the previous query)</small>

<!-- > -->

Use an **alias** to solve the problem!

```graphql
{
  rick: character(id: 1) {
    name
  }

  morty: character(id: 2) {
    name
  }
}
```

<small>(rick and morty are aliases — you could use any name!)</small>

<!-- > -->

The result would look like this:

```JS
{
  "data": {
    "rick": {
      "name": "Rick Sanchez"
    },
    "morty": {
      "name": "Morty Smith"
    }
  }
}
```

<small>(Results from the previous query)</small>

<!-- > -->

## After This Lesson

- Watch the videos here: https://www.howtographql.com
  - Introduction
  - GraphQL is the better REST
  - Core Concepts
  - Big Picture (Architecture)
- Complete the queries below and submit on GradeScope.
  - For each question write the GraphQL query that returns the requested data.

<!-- > -->

### Evaluate your Work

Submit on GradeScope. Write the **query** that solves each question. Use the Rick and Morty GraphQL API at https://rickandmortyapi.com/graphql.

1. Get Rick Sanchez's name and status.
1. Get Morty Smith's name, species, and gender.
1. Get Summer Smith's name and the name of her current location.
1. Get the total count of all characters. (Hint: try `characters { info { count } }`)
1. Get the name and air date of episode 1.
1. Get Rick's name and the name of his origin location.
1. Get the dimension of Rick's origin location.
1. Get both Rick and Morty's names and species using a **single query**. Use aliases!
1. Get both Rick's origin location name and Morty's origin location name using a single query. Use aliases!
1. Get the names of the first 3 residents of the Citadel of Ricks. (Hint: try `location(id: 3) { residents { name } }`)

<!-- > -->

## Additional Resources

- https://www.howtographql.com
- https://rickandmortyapi.com/graphql
- https://rickandmortyapi.com/documentation
