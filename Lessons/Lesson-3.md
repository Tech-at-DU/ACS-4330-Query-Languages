# ACS 4330 - GraphQL + Apollo Server

**Estimated time:** 3–4 hours

**Prerequisites:** Completed Lesson 2. You have a working Apollo Server with a schema and resolvers.

<!-- > -->

## Learning Outcomes

By the end of this lesson, you should be able to:

1. Build a GraphQL API that wraps a public REST API
1. Use Apollo Server with async resolvers
1. Define resolvers that fetch external data
1. Handle resolver arguments

<!-- > -->

## Review

Before starting, write out schema definitions for these types from memory. Check your work against your Lesson 2 server.

<!-- > -->

**Monster Battle**

Write the schema in the GraphQL Schema Language:
- `Kaiju` type — a giant monster (name, power)
- `City` type — (name, population)
- `Battle` type — includes two monsters and a city
- `Query` type — returns a battle

Then write a GraphQL query that fetches a battle result, printing monster names, powers, and the city name and population.

<!-- > -->

**Card Game**

Write the schema:
- `Card` type — playing card (value, suit)
- `Hand` type — a hand of cards
- `Query` type — returns a hand

Then write a query that gets a hand and prints the value and suit of each card.

<!-- > -->

**Users and Images**

Write the schema:
- `Location` type — latitude and longitude
- `Image` type — URL, size, and a location
- `User` type — includes a list of images
- `Query` type — returns a user

Then write a query that gets a user's images and prints the user name and image URLs.

<!-- > -->

## Overview 🌏

This lesson: build a GraphQL server that wraps a public REST API.

*Why?* A GraphQL layer over a REST API is a common real-world pattern. It lets clients query exactly what they need, even when the underlying data source is a REST API they don't control.

<!-- > -->

## GraphQL 😎 and Apollo Server

GraphQL is a specification and language — not a library. Libraries implement it.

**Apollo Server** is the most widely used GraphQL server library for Node.js.

<!-- > -->

### What's needed

- `@apollo/server` npm package
- `dotenv` npm package (for API keys)

<!-- > -->

### How do you set this up?

- Import Apollo Server
- Define your schema (`typeDefs`)
- Define your resolvers
- Start the server with `startStandaloneServer`

<!-- > -->

## Challenge

Build a GraphQL API that wraps the OpenWeatherMap REST API.

<small>Think of this as interview question practice — wrapping a REST API with GraphQL is a common task.</small>

<!-- > -->

You'll use https://openweathermap.org.

<small>Free, easy, good for a short assignment.</small>

<!-- > -->

**Challenge 1 - Setup Apollo Server**

<!-- > -->

1. Create a new folder
1. Initialize: `npm init -y`
1. Add `"type": "module"` to `package.json`
1. Install dependencies: `npm install @apollo/server dotenv`
1. Create `server.js`
1. Add `"start": "node --watch server.js"` to the `scripts` section of `package.json`

<!-- > -->

**Important!** Add a `.gitignore` — never commit `node_modules` or your `.env` file.

https://www.toptal.com/developers/gitignore/api/node

<!-- > -->

In `server.js` import your dependencies:

```js
import { ApolloServer } from '@apollo/server'
import { startStandaloneServer } from '@apollo/server/standalone'
import 'dotenv/config'
```

<!-- > -->

Start your schema:

```js
const typeDefs = `#graphql
  type Test {
    message: String!
  }

  type Query {
    test: Test
  }
`
```

<!-- > -->

Set up your resolvers:

```js
const resolvers = {
  Query: {
    // resolvers here
  }
}
```

<!-- > -->

Start your server:

```js
const server = new ApolloServer({ typeDefs, resolvers })

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 }
})

console.log(`Server ready at: ${url}`)
```

<!-- > -->

Run your server:

```bash
npm start
```

Open Apollo Sandbox: `http://localhost:4000`

<!-- > -->

**Challenge 2 - Get your API Key**

Go to https://openweathermap.org

1. Create an account at [openweathermap.org](https://openweathermap.org/).
1. After you make an account, click on your username, then on `My API Keys`. Enter an API key name, then click the `Generate` button to create your API key!

<!-- ![api_keys](../assets/api_keys.png) -->

<!-- > -->

**Quick Note on `.env` files**

A `.env` file stores secrets like API keys without exposing them on GitHub. You already installed `dotenv` in Challenge 1 and imported it with `import 'dotenv/config'`.

<!-- > -->

1. Create a `.env` file in your project folder
1. Add your API key (replace the placeholder):

```
OPENWEATHERMAP_API_KEY=__MY_API_KEY__
```

**Important:** Add `.env` to your `.gitignore` — never commit API keys.

<!-- > -->

Access your API key anywhere in `server.js` with:

```js
const apikey = process.env.OPENWEATHERMAP_API_KEY
```

<!-- > -->

**Challenge 3 - Define Schema**

Define your schema

```JS 
type Weather {
  temperature: Float!
  description: String!
}

type Query {
  getWeather(zip: Int!): Weather!
}
```

<small>Define a weather type and a query type to get the weather.</small>

<!-- > -->

**Challenge 4 - Define your Resolver**

Node 18+ includes `fetch` built-in — no package needed.

Define your resolver inside the `Query` object:

```js
const resolvers = {
  Query: {
    getWeather: async (_, { zip }) => {
      const apikey = process.env.OPENWEATHERMAP_API_KEY
      const url = `https://api.openweathermap.org/data/2.5/weather?zip=${zip}&appid=${apikey}`
      const res = await fetch(url)
      const json = await res.json()
      const temperature = json.main.temp
      const description = json.weather[0].description
      return { temperature, description }
    }
  }
}
```

<small>Apollo resolvers receive `(parent, args, context, info)`. Arguments from the query (like `zip`) come in the second parameter.</small>

<!-- > -->

**Challenge 5 - Test your work in Apollo Sandbox**

Try out a query and solve any errors that might pop up.

```JS
{
  getWeather(zip: 94010) {
    temperature
    description
  }
}
```

<!-- > -->

**Challenge 6 - Add units**

The weather API supports a unit of `standard`, `metric`, or `imperial`. Currently, you should be getting the weather in Kelvin (standard) this is hard to understand better to allow a request to include the unit. 

<!-- > -->

Add an enum for the type to your schema.

```JS
enum Units {
  standard
  metric
  imperial
}
```

<!-- > -->

Use the unit in your `getWeather` query.

```js
type Query {
  getWeather(zip: Int!, units: Units): Weather!
}
```

<!-- > -->

Handle the unit in your resolver:

```js
const resolvers = {
  Query: {
    getWeather: async (_, { zip, units = 'imperial' }) => {
      const apikey = process.env.OPENWEATHERMAP_API_KEY
      const url = `https://api.openweathermap.org/data/2.5/weather?zip=${zip}&appid=${apikey}&units=${units}`
      const res = await fetch(url)
      const json = await res.json()
      return { temperature: json.main.temp, description: json.weather[0].description }
    }
  }
}
```

<small>Be sure to add `units` to the query string!</small>

<!-- > -->

Test your work! Write a query: 

```JS
{
  getWeather(zip: 94122, units: metric) {
    temperature
    description
  }
}
```

<small>Notice that the enum value is NOT input as a string! Graphiql will code hint valid values! Go GraphQL introspection FTW!</small>

<!-- > -->

**Challenge 7 - Expand the API**

If you followed all of the instructions here your API should allow fetching the temperature and description. The OpenWeatherMap response provides a lot more information. The goal of this challenge is to expand the `getWeather` query type. 

<!-- > -->

Challenge, expand your query to include the following properties:

- feels_like
- temp_min
- temp_max
- pressure
- humidity

<!-- > -->

**Challenge 8 - Handle Errors**

The OpenWeatherMap API provides a cod property that includes an error code. If you provide a zipcode that doesn't exist you'll get a JSON object with a code of 404 and a message property with a message string. It might look something like: 

```JSON
{ cod: '404', message: 'city not found' }
```

Notice that `'404'` is a string. If you get a successful request the JSON will look like this: 

```JSON 
{ ..., cod: 200 }
```

<!-- > -->

When COD is 200 it's a number! 

Think about the results returned by your GraphQL API... What happens when you request this: 

```
{
  getWeather(zip:99999) {
    temperature
  }
}
```

99999 is not a valid zip the JSON object from OpenWeatherMap will include `"cod": "404"` and `"message":"city not found"`. All of the other information will be missing. 

<!-- > -->

Think about the data types defined in your getWeather query Type...

In this case, you won't have the temperature. But you will have a message. 

<!-- > -->

Your goal here is to return temperature, humidity, etc. sometimes, and include cod, and message sometimes. Don't overthink the solution (it may be easier than you first think). Talk it over with other students. 

Here's a clue: if you make a query for temperature with an invalid zip code then the temperature should be null!

<!-- > -->

Here's what this situation might look like in code.

The Query might look like this: 

```JS
{
  getWeather(zip:99999) {
    temperature
    description
    feels_like
    temp_min
    temp_max
    pressure
    humidity
    cod
    message
  }
}
```

<!-- > -->

The results would look like this: 

```JS
{
  "data": {
    "getWeather": {
      "temperature": null,
      "description": null,
      "feels_like": null,
      "temp_min": null,
      "temp_max": null,
      "pressure": null,
      "humidity": null,
      "cod": 404,
      "message": "city not found"
    }
  }
}
```

<!-- > -->

## Stretch Challenges 💪

<!-- > -->

Try as many of these stretch goals as you can!

- Expand the Weather API
  - Expand your the OpenWeatherMap other request parameters
    - timezone
    - name
    - id
    - visibility
    - wind: speed, deg, gust
    - coord: lat and lon
    - clouds: all
    - dt
    - sys: type, country, id, sunrise, sunset
    - main: sea_level, grnd_level
  - Currently, your API supports zip code but the current weather forecast supports
    - city name
    - city id
    - latitude and longitude
  - The example above uses the Current Weather API. OpenWeathermap also provides several other APIs that you can use. Make your GraphQL server support one of these: 
  - [Minute Forecast 1 hour*](https://openweathermap.org/api/one-call-api)
  - [Hourly Forecast 2 days*](https://openweathermap.org/api/one-call-api)
  - [Daily Forecast 7 days*](https://openweathermap.org/api/one-call-api)
  - [National Weather Alerts*](https://openweathermap.org/api/one-call-api)
  - [Historical weather 5 days*](https://openweathermap.org/api/one-call-api)
- Build a GrpahQL API on top of another API (you choose the API)

<!-- > -->

## After This Lesson

- Submit your completed GraphQL Weather API to GradeScope.
  - Solve as many challenges as you can!
  - If you finish all challenges, try the stretch challenges.

<!-- > -->

### Evaluate your work

1. Build a GraphQL API over a public REST API
1. Use Apollo Server with async resolvers
1. Define resolvers that fetch external data

| - | Does not meet expectations | Meets Expectations | Exceeds Expectations |
|:---:|:---:|:---:|:---:|
| Comprehension of Resolvers | Can't explain what a resolver is | Can describe what a resolver does and find it in the code | Could describe how resolvers could be used beyond this example |
| Apollo Server setup | Can't set up a basic Apollo Server | Could set up Apollo Server with a schema and resolvers | Could teach another student how to set up Apollo Server |
| Async Resolvers | Can't write an async resolver | Can write an async resolver using native `fetch` | Could expand the resolver to handle errors and additional fields |

<!-- > -->

## Resources

- https://www.toptal.com/developers/gitignore/api/node
- https://www.apollographql.com/docs/apollo-server/getting-started
- https://openweathermap.org/api

