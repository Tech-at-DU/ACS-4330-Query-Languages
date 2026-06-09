# ACS 4330 - React + Apollo Client

**Estimated time:** 4–5 hours

**Prerequisites:** Completed Lesson 3. Your GraphQL weather server is working. Basic JavaScript (ES modules, async/await).

<!-- > -->

Your GraphQL server needs a frontend!

<!-- > -->

Any front-end client can connect to a GraphQL server. This course uses React.

<!-- > -->

React is a library for building user interfaces — one of the most widely used in industry.

<!-- > -->

## Learning Objectives

1. Build a React app with Vite
2. Create reusable components
3. Use JSX, State, and Props
4. Use Hooks
5. Connect React to a GraphQL server with Apollo Client

<!-- > -->

## Overview

<!-- > -->

The React library has several core features let's take a look at those:

- Components
- JSX
- ReactDOM

<!-- > -->

## Creating a React App

<!-- > -->

Create a React app using Vite:

```bash
npm create vite@latest weather-client -- --template react
cd weather-client
npm install
npm run dev
```

This creates a new folder with a complete React project and starts a dev server at `http://localhost:5173`.

<!-- > -->

Tour the project:

- `public/`
- `src/`
  - `main.jsx` — entry point
  - `App.jsx` — root component
  - `index.css`
  - `App.css`

<!-- > -->

### Components

<!-- > -->

Components are the foundational building block of React Applications. Most often a component represents a view.

<!-- > -->

Components are composable. Components can be nested within other components. Complex components are made from smaller less complex components.

- Components must return [JSX](#jsx)
- Components can be built from a function or a class

<!-- > -->

This is a Component

```JS
function Header(props) {
  return (
    <h1>{props.title}</h1>
  )
}
```

<small>In it's simplest form a Component is a function that returns JSX.</small>

<!-- > -->

What's JSX? 

JSX is an extension of the JavaScript Language. 

JSX === JavaScript and XML. 

<!-- > -->

JSX provides a HTML like syntax that compiles to standard JS. For example: 

```js
<h1>{props.title}</h1>
```

Becomes: 

```JS
React.createElement("h1", null, props.title);
```

<small>The magic is that the first line looks exactly like what will be generated for the browser to display.</small> 

<!-- > -->

Why use JSX? 

- Looks like the HTML will generate
- Easier to reason about

<!-- > -->

## JSX has its own rules! 

<!-- > -->

1. Must have a top-level node

```JS
// Good!
<div>
  <h1>Hello</h1>
  <p>World</p>
</div>
```

```JS
// Error!
<h1>Hello</h1>
<p>World</p>
```

<!-- > -->

If you don't want to create an extra tag use a fragment!

```JS
// Good!
<>
  <h1>Hello</h1>
  <p>World</p>
</>
```

<!-- > -->

Can't use `class`, use `className` instead!

```JS
// Good!
<div className="App">
  <h1 className="title">Hello</h1>
  <p>World</p>
</div>
```

```JS
// Error!
<div class="App">
  <h1 class="title">Hello</h1>
  <p>World</p>
</div>
```

<!-- > -->

A tag with no children must be self-closing by adding a `/` be the closing `>`.

```JS
// Error!
<div class="App">
  <br />
  <img />
  <hr />
  <Game />
  <Map />
</div>
```

<!-- > -->

Values in JSX are strings (use double quotes!) Other values use `{}`.

```JS
<div>
  <img src={url} width={100} height={150} />
</div>
```

<small>Imagine `url` is a variable</small>

<!-- > -->

Any JS expression can be used in a JSX expression as long as it appears in the `{}`.

```JS
<div>
  <img 
    src={`${path}/${filename}`} 
    width={w * 0.5} 
    height={h * 0.5} 
    onClick={() => {
      ...
    }}
  />
</div>
```

<small>Move attributes to their own line for clarity.</small>

<!-- > -->

### Composing Components

<!-- > -->

Store components each in their file. 

```JS
// Title.js

function Title() {
  return <h1>Hello World</h1>
}

export default Title
```

```JS
// App.js

import Title from './Title.js'

function App() {
  return <Title />
}
```

<!-- > -->

If you're returning more than one line of JSX wrap in `()`.

```JS
// App.js
import Title from './Title.js'

function App() {
  return ( // <-- (
    <div className="App">
      <Title />
    </div>
  ) // <-- )
}
```

<!-- > -->

## Props

<!-- > -->

Props are values passed to a component. 

When props change the component renders. 

Props come from outside the component and are passed into the component.

<!-- > -->

Pass props to a component via attributes: 

```JS
// Imagine we're using the component somewhere
<Heading title="Hello" subtitle="world" />
```

```JS
// Heading.js
function Heading(props) {
  // { title: "Hello", subtitle: "world" }
  return (
    <>
      <h1>{props.title}</h1>
      <p>{props.subtitle}</p>
    </>
  )
} 
```

<!-- > -->

## State 

<!-- > -->

State presents values component stores internally. 

When the state changes the component renders. 

<!-- > -->

Define state like this: 

```JS
import { useState } from 'react'

function Counter() {
  const [ count, setCount ] = useState(0)
  return (
    <>
      <h1>{count}</h1>
      <button 
        onClick={() => setCount(count + 1)}
      >Add 1</button>
    </>
  )
}
```

<small>Here `count` is increased by 1 with each click.</small>

<!-- > -->

## Controlled Component Pattern

<!-- > -->

The controlled component pattern is used to handle form elements. 

An input value is stored as state.

<!-- > -->

Controlled component pattern in action!

```JS
import { useState } from 'react'

function Counter() {
  const [ zip, setZip ] = useState('')
  return (
    <>
    <input 
      value={zip}
      onChange={(e) => setZip(e.target.value)}
    />
    </>
  )
}
```

<small>Here state holds the value set on the input and is updated when the input changes.</small>

<!-- > -->

## Inputs and forms

<!-- > -->

Forms are really important. They have some behaviors that can make them a little confusing. 

<!-- > -->

Group all of your form elements in a form!

```JS
<form>
  <p>Send us a message</p>
  <input type="email" />
  <textarea></textarea>
  <label>
    Sign up for out news letter!
    <input type="checkbox" />
  </label>
  ...
</form>
```

<!-- > -->

Submit a form with a submit button!

```JS
<form>
  ...
  <button type="submit">Submit</button>
</form>
```

<small>A button with `type="submit" will submit a form!`</small>

<!-- > -->

Handle submitting a form with `onSubmit`

```JS
<form onSubmit={(e) => {
  // Handle your form submission here!
}}>
  ...
</form>
```

<small>Like other event handlers `onSubmit` receives an event object! (`e` in the sample above)</small>

<!-- > -->

Submitting a form will reload the current page, you need to prevent this in a React App!

```JS
<form onSubmit={(e) => {
  e.preventDefault() // prevent the page from loading!
  ...
}}>
  ...
</form>
```

<!-- > -->

## Connecting a Client to GraphQL

<!-- > -->

For the client-side, you'll be using Apollo GraphQL client.

<!-- > -->

The goal of this project is to create a client built with React that connects to your GraphQL OpenWeatherMap server. 

<!-- > -->

The server runs at `localhost:4000` and the React client runs at `localhost:5173`.

You need to start **both** for the project to work locally — open two terminals.

<!-- > -->

Keep the two projects in separate folders.

<!-- > -->

## Apollo Client

<!-- > -->

**Apollo Client** is a comprehensive state management library for JavaScript that enables you to manage both local and remote data with GraphQL. Use it to fetch, cache, and modify application data, all while automatically updating your UI.

<!-- > -->

You will use Apollo Client in this project to handle transactions between your React project and your GraphQL server. 

<!-- > -->

**CORS Note:** Apollo Server 4 standalone mode allows all origins by default in development. No separate CORS configuration needed for local development.

<!-- > -->

**What's CORS?**

Cross-Origin Resource Sharing — HTTP headers that control which origins can access a server. Your React app runs on `localhost:5173` and your GraphQL server on `localhost:4000`, so they're different origins.

<!-- > -->

Follow these steps to set up Apollo Client with React.

The React project was already created in the React setup above. Now install Apollo Client:

```bash
npm install @apollo/client graphql
```

<!-- > -->

Create a dedicated file `src/apolloClient.js` for the Apollo Client instance:

```js
import { ApolloClient, InMemoryCache } from '@apollo/client'

export const client = new ApolloClient({
  uri: 'http://localhost:4000/',
  cache: new InMemoryCache()
})
```

<small>Apollo Server 4 standalone serves GraphQL at the root `/` — not `/graphql`.</small>

<!-- > -->

In `src/main.jsx`, wrap your app in `<ApolloProvider>`:

```js
import React from 'react'
import ReactDOM from 'react-dom/client'
import { ApolloProvider } from '@apollo/client'
import { client } from './apolloClient'
import App from './App'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <ApolloProvider client={client}>
      <App />
    </ApolloProvider>
  </React.StrictMode>
)
```

<!-- > -->

Now you can use Apollo in any component under `<App>`. Import `client` and `gql`:

```js
import { gql } from '@apollo/client'
import { client } from './apolloClient'
```

<!-- > -->

Now you're ready to make requests to your GraphQL server from your React project. 

- Here are a few ideas: 
  - The requests are async. You will need to handle them with Promise. 
  - You will need to render your components conditionally. 
  - Store the data you load on state. This will cause your components to render when state changes. 

<!-- > -->

Make a component to handle a request to the weather server. 

```jsx
import { useState } from 'react'
import { gql } from '@apollo/client'
import { client } from './apolloClient'

function Weather() {
  return (
    <div className="Weather">
      <form>
        ...
      </form>
    </div>
  )
}

export default Weather
```

<!-- > -->

Add state to handle the form input and the weather data. 

```JS
function Weather() {
  const [ zip, setZip ] = useState('')
  const [ weather, setWeather ] = useState(null)

  return (
    <div className="Weather">
      <form>
        ...
      </form>
    </div>
  );
}
```

<!-- > -->

Add an input and submit button.

```JS
<div className="Weather">
  <form>
    <input 
      value={zip}
      onChange={(e) => setZip(e.target.value)}
    />
    <button type="submit">Submit</button>
  </form>
</div>
```

<!-- > -->

Add a function to handle requests to your GraphQL server.

```js
function Weather() {
  ...
  async function getWeather() {
    try {
      const json = await client.query({
        query: gql`
          query GetWeather($zip: Int!) {
            getWeather(zip: $zip) {
              temperature
              description
            }
          }
        `,
        variables: { zip: parseInt(zip, 10) }
      })
      setWeather(json)
    } catch(err) {
      console.log(err.message)
    }
  }
  ...
}
```

<small>Use query variables (`$zip`) instead of string interpolation — cleaner and avoids injection issues.</small>

<!-- > -->

Handle the submit event: 

```js
<form onSubmit={(e) => {
  e.preventDefault()
  getWeather()
}}>
  <input 
    value={zip}
    onChange={(e) => setZip(e.target.value)}
  />
  <button type="submit">Submit</button>
</form>
```

<!-- > -->

Handle displaying your weather data. 

```JS
<div className="Weather">

  {weather ? <h1>{weather.data.getWeather.temperature}</h1>: null}

  <form onSubmit={(e) => {
    e.preventDefault()
    getWeather()
  }}>
  ...
  </form>
</div>
```

<small>This conditional rendering technique looks at `weather`, if its truthy displays the h1, otherwise `null`</small>

<!-- > -->

All of the examples here may need some name changes to work with your server and components!

<!-- > -->

## Challenges

<!-- > -->

You will build a React App that fetches weather data from your GraphQL Weather server. It will be used by the Apollo client for GraphQL for queries.

<!-- > -->

**Challenge 0 - Weather Server**

Complete your GraphQL Apollo weather server from Lesson 3.

- Apollo Server 4 standalone handles CORS automatically — no extra setup needed.

<!-- > -->

**Challenge 1 - Create your React Project**

Follow the Vite instructions above and confirm the app loads at `http://localhost:5173`.

<!-- > -->

**Challenge 2 - Run your GraphQL Server**

Your Apollo server runs on port 4000, the React client on port 5173.

- Start both: open two terminal tabs and run `npm start` in each project folder.

<!-- > -->

**Challenge 3 - Add Apollo Client**

Add Apollo Client to your React project and create `src/apolloClient.js`. Follow the instructions above.

- URI must be `'http://localhost:4000/'` — Apollo Server 4 serves GraphQL at root, not `/graphql`.

<!-- > -->

**Challenge 4 - Create a Component**

This component will connect to the GraphQL Server. See the instructions above. 

- Import `gql` and `client`
- Set up your form and an async function to handle requests

<!-- > -->

**Challenge 5 - Handle your data with conditional rendering**

Handle data with conditional rendering. See the instructions above. 

The goal here is to display data after it's loaded and display nothing or alternate content when no data is available.

- Display the temperature

<!-- > -->

**Challenge 6 - Make a component to display the Weather**

React is all about components. Write a specialized component that is used just to display the weather data. 

Supply data as props. Display: 

- temp
- description
- and three or more properties!

<!-- > -->

**Challenge 7 - Handle Errors**

If you enter a zip code that doesn't exist you'll get an error or nothing will happen. You should handle this situation. 

- Check for an error, use the cod and message value from openweather map. Your Server should provide this! 
- Display the message when cod != 200.

<!-- > -->

**Challenge 8 - Style Your work**

This is an open-ended challenge. Do as much work here as you like!

- Style the form
- Style the weather results
- Display an image for the weather

<!-- > -->

**Challenge 9 - Use Units**

Add a radio button or another method to set the unit type: metric or imperial. And display the weather with that unit. 

- Add two radio buttons to the UI. 
- Send the unit to the server
- Server should request data in the unit from openweathermap

<!-- > -->

**Challenge 10 - Get the current location**

OpenWeatherMap supports getting the weather by location. You can get the geocoordinates with the browser API. 

Make a button that gets the coordinates. 

- https://developer.mozilla.org/en-US/docs/Web/API/Navigator/geolocation

<!-- > -->

**Challenge 11 - Add Weather by location to your GraphQL API**

Spends some time on your GraphQL server. Add a query type that gets weather by geolocation. 

<!-- > -->

**Challenge 12 - Get weather by geolocation**

Add a button to your React project that gets the geolocation from the browser and makes a request to the GraphQL API. 

<!-- > -->

**Challenge 13 - Sub another API**

This is an open-ended stretch challenge. Substitute another API for the OpenWeatherMap API.

<!-- > -->

## After This Lesson

Complete the challenges above and submit your work to GradeScope.

<!-- > -->

## Resources

- [React Docs](https://react.dev)
- [Hooks](https://react.dev/reference/react)
- [Async/await](https://javascript.info/async-await)
- [Apollo Client React Docs](https://www.apollographql.com/docs/react/get-started/)
- [Vite Getting Started](https://vitejs.dev/guide/)

<!-- > -->
