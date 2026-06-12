# ACS 4330 - Query Variables and Apollo Hooks

**Estimated time:** 3–4 hours

**Prerequisites:** Completed Lessons 4 and 6. Working React + Apollo Client weather app with a form and mutations.

<!-- > -->

## Learning Outcomes

By the end of this lesson, you should be able to:

1. Write GraphQL queries using variables instead of hardcoded values
2. Pass variables to queries in Apollo Sandbox
3. Refactor a React component from `useQuery` to `useLazyQuery`
4. Identify the right Apollo hook for a given use case

<!-- > -->

## Review

Before starting, write answers from memory:

- What is the difference between a query and a mutation?
- What arguments does a resolver receive? What's in the second argument?
- In Apollo Client, what does `useQuery` return?

<!-- > -->

## 📦 Variables in GraphQL

So far your queries have hardcoded values:

```graphql
{
  getWeather(zip: 94122, units: imperial) {
    temperature
    description
  }
}
```

Variables make queries **dynamic** — the query structure stays the same, the values change.

<!-- > -->

### 💡 Why Use Variables?

- **Reusability** — same query, different values
- **Security** — user input is passed separately, not embedded in the query string
- **Performance** — GraphQL engines can cache the query structure when variables are separate
- **Readability** — query logic stays clean

<!-- > -->

### Syntax

A query with variables has three parts:

**1. Declare the variable in the operation name**

```graphql
query GetWeather($zip: Int!, $units: Units) {
```

**2. Use the variable in the query**

```graphql
  getWeather(zip: $zip, units: $units) {
```

**3. Pass values as a JSON object**

```json
{
  "zip": 94122,
  "units": "imperial"
}
```

<!-- > -->

Full example:

```graphql
query GetWeather($zip: Int!, $units: Units) {
  getWeather(zip: $zip, units: $units) {
    temperature
    description
  }
}
```

Variables:
```json
{
  "zip": 94122,
  "units": "imperial"
}
```

<!-- > -->

### 🏷️ Variable Types

| Type | Description | Example |
|------|-------------|---------|
| `Int` | Whole number | `42` |
| `Float` | Decimal number | `3.14` |
| `String` | Text | `"Hello"` |
| `Boolean` | True or false | `true` |
| `ID` | Unique identifier | `"a1234"` |
| `[Type]` | List | `[1, 2, 3]` |
| `Type!` | Required (non-nullable) | `Int!` |

<!-- > -->

## Challenge 1 — Variables in Apollo Sandbox

Open Apollo Sandbox at `http://localhost:4000` with your weather server running.

1. Write a `query` with an operation name and `$zip` and `$units` variables
2. Open the **Variables** panel (bottom of the editor)
3. Paste your variables as JSON
4. Run the query and verify results

Try changing the zip and units without editing the query — just change the JSON variables.

<!-- > -->

## 🪝 Variables in Apollo Client

### `useQuery` with Variables

Pass variables as the second argument to `useQuery`:

```jsx
import { gql, useQuery } from '@apollo/client'

const GET_WEATHER = gql`
  query GetWeather($zip: Int!, $units: Units) {
    getWeather(zip: $zip, units: $units) {
      temperature
      description
    }
  }
`

function WeatherComponent({ zip }) {
  const { loading, error, data } = useQuery(GET_WEATHER, {
    variables: { zip, units: 'imperial' },
  })

  if (loading) return <p>Loading...</p>
  if (error) return <p>Error: {error.message}</p>

  return (
    <div>
      <p>Temperature: {data.getWeather.temperature}°F</p>
      <p>Description: {data.getWeather.description}</p>
    </div>
  )
}
```

`useQuery` fires immediately when the component mounts and re-runs when `variables` change.

<!-- > -->

### 🦥 `useLazyQuery` — Fetch on Demand

`useLazyQuery` is identical to `useQuery` except it **does not fire on mount**. You trigger it manually — perfect for a form where the user types a zip and clicks a button.

```jsx
const [executeQuery, { loading, error, data }] = useLazyQuery(QUERY)
```

Compare:

| | `useQuery` | `useLazyQuery` |
|---|---|---|
| When it runs | On mount, and when variables change | Only when you call it |
| Return value | `{ loading, error, data }` | `[executeFn, { loading, error, data }]` |
| Use case | Data always needed | Data fetched on user action |

<!-- > -->

Full `useLazyQuery` weather example:

```jsx
import { gql, useLazyQuery } from '@apollo/client'
import { useState } from 'react'

const GET_WEATHER = gql`
  query GetWeather($zip: Int!, $units: Units) {
    getWeather(zip: $zip, units: $units) {
      temperature
      description
    }
  }
`

function WeatherComponent() {
  const [zip, setZip] = useState('')
  const [getWeather, { loading, error, data }] = useLazyQuery(GET_WEATHER)

  const handleFetch = () => {
    const zipCode = parseInt(zip, 10)
    if (isNaN(zipCode)) return
    getWeather({ variables: { zip: zipCode, units: 'imperial' } })
  }

  return (
    <div>
      <input
        type="number"
        value={zip}
        onChange={(e) => setZip(e.target.value)}
        placeholder="ZIP code"
      />
      <button onClick={handleFetch}>Get Weather</button>

      {loading && <p>Loading...</p>}
      {error && <p>Error: {error.message}</p>}
      {data && (
        <div>
          <p>Temperature: {data.getWeather.temperature}°F</p>
          <p>Description: {data.getWeather.description}</p>
        </div>
      )}
    </div>
  )
}

export default WeatherComponent
```

<!-- > -->

### Common Mistakes

`useLazyQuery` returns `undefined` until triggered:

```js
const [getWeather, { data }] = useLazyQuery(GET_WEATHER)
console.log(data) // undefined — query hasn't run yet
```

Pass zip as a number, not a string:

```js
// Wrong — zip is a string from input.value
getWeather({ variables: { zip } })

// Right
getWeather({ variables: { zip: parseInt(zip, 10) } })
```

<!-- > -->

## Challenge 2 — Refactor to `useLazyQuery`

Refactor your React weather component to use `useLazyQuery`.

1. Replace the `useQuery` import with `useLazyQuery`
2. Update the destructuring: `const [getWeather, { loading, error, data }] = ...`
3. Remove any automatic-fetch behavior — query should only run when the button is clicked
4. Call `getWeather({ variables: { zip: parseInt(zip, 10) } })` in your submit handler
5. Test: load the page, confirm no query fires until you submit the form

<!-- > -->

## 🧩 Fragments

Fragments let you reuse field selections across multiple queries.

```graphql
fragment CharacterInfo on Character {
  name
  status
  species
}

query {
  rick: character(id: 1) {
    ...CharacterInfo
  }
  morty: character(id: 2) {
    ...CharacterInfo
  }
}
```

Try this at https://rickandmortyapi.com/graphql.

Without the fragment you would repeat `name status species` for every character. Fragments keep queries DRY.

<!-- > -->

## Challenge 3 — Fragments in Apollo Sandbox

1. Open https://rickandmortyapi.com/graphql
2. Define a fragment on `Character` with at least 3 fields
3. Write a query that fetches 3 different characters using your fragment
4. Verify all 3 return the same fields

<!-- > -->

## Apollo Hooks Reference

Quick reference for all Apollo Client hooks.

<!-- > -->

### 🪝 Data Hooks

| Hook | Runs when | Use case |
|------|-----------|---------|
| `useQuery` | Component mounts | Data always needed |
| `useLazyQuery` | You call it | Forms, search, on-demand |
| `useMutation` | You call it | Create, update, delete |
| `useSubscription` | Server pushes | Real-time updates |

<!-- > -->

### ⚡ `useQuery`

Fetches data immediately on mount. Re-fetches automatically when `variables` change.

```jsx
const { loading, error, data } = useQuery(GET_USER, {
  variables: { id: userId },
})
```

<!-- > -->

### 🦥 `useLazyQuery`

Same as `useQuery` but waits for you to call it.

```jsx
const [searchUser, { loading, data }] = useLazyQuery(SEARCH_USER)

<button onClick={() => searchUser({ variables: { name: 'Rick' } })}>
  Search
</button>
```

<!-- > -->

### ✏️ `useMutation`

Executes a mutation. Returns a trigger function and status object.

```jsx
const [addPet, { loading, error }] = useMutation(ADD_PET)

<button onClick={() => addPet({ variables: { name: 'Snuffles', species: 'cat' } })}>
  Add Pet
</button>
```

<!-- > -->

### 📡 `useSubscription`

Listens for real-time updates over WebSocket.

```jsx
const { data } = useSubscription(MESSAGE_RECEIVED)

return <p>New message: {data?.messageReceived.text}</p>
```

Covered in detail in Lesson 11 (Subscriptions).

<!-- > -->

### 🔑 `useApolloClient`

Direct access to the Apollo Client instance — useful for manually resetting the cache.

```jsx
const client = useApolloClient()

<button onClick={() => client.resetStore()}>Clear Cache</button>
```

<!-- > -->

### 🧩 `useFragment`

Reads a specific fragment from the Apollo cache without re-fetching.

```jsx
const user = useFragment({
  fragment: USER_FRAGMENT,
  fragmentName: 'UserFragment',
  id: `User:${id}`,
})
```

**Best for:** Reading specific fields already in cache without a full query.

<!-- > -->

### 🔁 `useReactiveVar`

Manages local state inside Apollo Client — no Redux or React Context needed.

```jsx
import { makeVar, useReactiveVar } from '@apollo/client'

const themeVar = makeVar('light')

function ThemeSwitcher() {
  const theme = useReactiveVar(themeVar)

  return (
    <button onClick={() => themeVar(theme === 'light' ? 'dark' : 'light')}>
      Theme: {theme}
    </button>
  )
}
```

<!-- > -->

## After This Lesson

- Submit your refactored React weather component (using `useLazyQuery` and variables) to GradeScope.

<!-- > -->

### Evaluate Your Work

| | Does not meet expectations | Meets expectations | Exceeds expectations |
|:---:|:---:|:---:|:---:|
| Variables | Can't write a query with variables | Writes queries with variables and passes them as JSON | Explains why variables are better than hardcoded values |
| `useLazyQuery` | Can't distinguish from `useQuery` | Refactors a component to use `useLazyQuery` | Could explain when to use each hook and why |
| Fragments | Can't read a fragment | Writes a fragment and uses it in a query | Uses fragments in Apollo Client with `useFragment` |

<!-- > -->

## Resources

- https://www.apollographql.com/docs/react/api/react/hooks
- https://graphql.org/learn/queries/#variables
- https://graphql.org/learn/queries/#fragments
- https://rickandmortyapi.com/graphql
