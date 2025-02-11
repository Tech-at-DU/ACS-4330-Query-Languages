# Variables in GraphQL

## 📌 Variables in GraphQL Query Language
Variables in GraphQL are placeholders that allow queries to be **dynamic** and **reusable**, instead of hardcoding values directly into queries. This makes GraphQL queries **more flexible, efficient, and secure**.

---

## 🔹 Why Use Variables?
1. **Reusability** – Run the same query with different values.
2. **Security** – Prevents hardcoding user inputs, reducing injection risks.
3. **Performance** – Allows GraphQL engines to cache query structures and only change values.
4. **Readability** – Keeps queries clean and easy to maintain.

---

## 🔹 Defining Variables in a Query
A GraphQL query with variables consists of **three parts**:
1. **Declare the variable in the query definition** (`$variableName: Type`).
2. **Use the variable in the query**.
3. **Pass values into the query** when executing it.

### Example 1: Query with a Variable
#### 🔸 GraphQL Query (Client-Side)
```graphql
query GetWeather($zip: Int!) {
  getWeather(zip: $zip, units: "imperial") {
    temperature
    description
  }
}
```
- `$zip` is a **variable** of type `Int!` (a required integer).
- It is passed to the `getWeather` query.

#### 🔸 Variables Passed to the Query
```json
{
  "zip": 90210
}
```

---

## 🔹 Variable Types in GraphQL
| Type  | Description | Example |
|--------|------------|---------|
| `Int`  | Integer (whole number) | `42` |
| `Float` | Decimal number | `3.14` |
| `String` | Text | `"Hello"` |
| `Boolean` | `true` or `false` | `true` |
| `ID` | Unique identifier | `"a1234"` |
| `[Type]` | List (array) | `[1, 2, 3]` |
| `Type!` | Non-nullable type (required) | `Int!` |


## Challenge! 

Using your GraphQL Weather Server write a couple queries **using variables**. 
- In the Graphiql browser you must define your variables in the **"Variables"** tab in the lower left of the window.
- When writing variables you might write them as **JSON**.

---

## 🔹 Passing Variables in Apollo Client (`useQuery` and `useLazyQuery`)
In a React app using Apollo Client, you pass variables when executing the query.

### 🔸 Example in Apollo Client (`useQuery`)
```jsx
import { gql, useQuery } from '@apollo/client';

const GET_WEATHER = gql`
  query GetWeather($zip: Int!) {
    getWeather(zip: $zip, units: "imperial") {
      temperature
      description
    }
  }
`;

const WeatherComponent = ({ zip }) => {
  const { loading, error, data } = useQuery(GET_WEATHER, {
    variables: { zip }, // Pass the variable here
  });

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <div>
      <p>Temperature: {data.getWeather.temperature}°F</p>
      <p>Description: {data.getWeather.description}</p>
    </div>
  );
};
```

---

### 🔸 Example with `useLazyQuery` (Fetching on Demand)
```jsx
import { gql, useLazyQuery } from '@apollo/client';
import { useState } from 'react';

const GET_WEATHER = gql`
  query GetWeather($zip: Int!) {
    getWeather(zip: $zip, units: "imperial") {
      temperature
      description
    }
  }
`;

function WeatherComponent() {
  const [zip, setZip] = useState('');
  const [getWeather, { loading, error, data }] = useLazyQuery(GET_WEATHER);

  const handleFetch = () => {
    const zipCode = parseInt(zip, 10);
    if (isNaN(zipCode)) {
      alert("Enter a valid ZIP code");
      return;
    }
    getWeather({ variables: { zip: zipCode } });
  };

  return (
    <div>
      <input 
        type="number" 
        value={zip}
        onChange={(e) => setZip(e.target.value)}
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
  );
}

export default WeatherComponent;
```
---

## 🔹 Best Practices for Using Variables in GraphQL
✅ **Use meaningful variable names** (`$userId` instead of `$id`).  
✅ **Use non-nullable (`!`) variables where required** (`Int!` ensures a valid input).  
✅ **Validate user input before passing variables** to avoid errors.  
✅ **Pass structured objects for complex queries** instead of multiple separate variables.

---

### 🔹 Summary
- **Variables make queries reusable, secure, and performant**.
- **Defined using `$variableName: Type` syntax**.
- **Passed separately in a JSON object** to the query execution.
- **Used in Apollo Client (`useQuery`, `useLazyQuery`) to fetch dynamic data**.

## Apollo Client hooks 

### **📌 Apollo Client Hooks Overview**
Apollo Client provides a set of powerful **React hooks** to handle GraphQL queries, mutations, and cache interactions efficiently.

---

## **🔹 Hooks for Data Fetching**
| Hook | Purpose |
|------|---------|
| [`useQuery`](#-usequery) | Fetches GraphQL queries when a component mounts and re-fetches on dependency updates. |
| [`useLazyQuery`](#-uselazyquery) | Fetches queries **on demand**, instead of on component mount. |
| [`useMutation`](#-usemutation) | Executes **mutations** to modify server data (e.g., create, update, delete). |
| [`useSubscription`](#-usesubscription) | Subscribes to **real-time updates** from the GraphQL server. |

---

## **🔹 Hooks for Apollo Cache & Client Interaction**
| Hook | Purpose |
|------|---------|
| [`useApolloClient`](#-useapolloclient) | Access the **Apollo Client instance** directly to perform cache updates or execute raw queries. |
| [`useFragment`](#-usefragment) | Reads a **GraphQL fragment** from the cache. |
| [`useReactiveVar`](#-usereactivevar) | Reads a **reactive variable** for local state management. |

---

## **🔍 Detailed Explanation of Each Hook**
### **1️⃣ `useQuery` (Auto-fetching GraphQL Queries)**
Fetches data **immediately when a component mounts** and re-fetches automatically when dependencies change.

#### ✅ **Example:**
```jsx
import { gql, useQuery } from '@apollo/client';

const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      name
      email
    }
  }
`;

function UserProfile({ userId }) {
  const { loading, error, data } = useQuery(GET_USER, {
    variables: { id: userId },
  });

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return <p>User: {data.user.name}</p>;
}
```
**Best for:** Fetching data automatically when the component mounts.

---

### **2️⃣ `useLazyQuery` (On-Demand Query Execution)**
Similar to `useQuery`, but **does not fetch data immediately**. Instead, it waits until a function is called.

#### ✅ **Example:**
```jsx
import { gql, useLazyQuery } from '@apollo/client';

const SEARCH_USER = gql`
  query SearchUser($name: String!) {
    searchUser(name: $name) {
      id
      name
    }
  }
`;

function SearchComponent() {
  const [searchUser, { loading, data }] = useLazyQuery(SEARCH_USER);
  
  return (
    <div>
      <button onClick={() => searchUser({ variables: { name: "Alice" } })}>
        Search User
      </button>
      {loading && <p>Loading...</p>}
      {data && <p>Found: {data.searchUser.name}</p>}
    </div>
  );
}
```
**Best for:** Fetching data when a user clicks a button or submits a form.

---

### **3️⃣ `useMutation` (Executing GraphQL Mutations)**
Used to **modify server data** (e.g., create, update, delete).

#### ✅ **Example:**
```jsx
import { gql, useMutation } from '@apollo/client';

const ADD_USER = gql`
  mutation AddUser($name: String!, $email: String!) {
    addUser(name: $name, email: $email) {
      id
      name
    }
  }
`;

function CreateUser() {
  const [addUser, { loading, error }] = useMutation(ADD_USER);

  const handleAddUser = () => {
    addUser({ variables: { name: "Alice", email: "alice@example.com" } });
  };

  return (
    <div>
      <button onClick={handleAddUser}>Add User</button>
      {loading && <p>Saving...</p>}
      {error && <p>Error: {error.message}</p>}
    </div>
  );
}
```
**Best for:** Sending **POST, PUT, DELETE** requests to modify data.

---

### **4️⃣ `useSubscription` (Real-Time Data Streaming)**
Subscribes to real-time **WebSocket updates** from a GraphQL server.

#### ✅ **Example:**
```jsx
import { gql, useSubscription } from '@apollo/client';

const MESSAGE_SUBSCRIPTION = gql`
  subscription OnMessageReceived {
    messageReceived {
      id
      text
    }
  }
`;

function Chat() {
  const { data } = useSubscription(MESSAGE_SUBSCRIPTION);

  return <p>New Message: {data?.messageReceived.text}</p>;
}
```
**Best for:** Live chat, notifications, stock prices, or any **real-time updates**.

---

### **5️⃣ `useApolloClient` (Direct Client Access)**
Gives access to the Apollo Client instance **without hooks like `useQuery`**.

#### ✅ **Example:**
```jsx
import { useApolloClient } from '@apollo/client';

function ClearCacheButton() {
  const client = useApolloClient();

  const handleClearCache = () => {
    client.resetStore();
  };

  return <button onClick={handleClearCache}>Clear Cache</button>;
}
```
**Best for:** Accessing Apollo cache, performing manual queries, or resetting the store.

---

### **6️⃣ `useFragment` (Reading GraphQL Fragments)**
Used to **read a fragment** from the Apollo cache.

#### ✅ **Example:**
```jsx
import { gql, useFragment } from '@apollo/client';

const USER_FRAGMENT = gql`
  fragment UserFragment on User {
    id
    name
  }
`;

const UserComponent = ({ id }) => {
  const user = useFragment({
    fragment: USER_FRAGMENT,
    fragmentName: "UserFragment",
    id: `User:${id}`,
  });

  return <p>{user?.name}</p>;
};
```
**Best for:** Reading **specific fields** from the cache instead of fetching an entire query.

---

### **7️⃣ `useReactiveVar` (Local State with Apollo Reactive Variables)**
Used to manage **local state** within Apollo Client **without using React state or context**.

#### ✅ **Example:**
```jsx
import { makeVar, useReactiveVar } from '@apollo/client';

// Create a global reactive variable
const themeVar = makeVar("light");

function ThemeSwitcher() {
  const theme = useReactiveVar(themeVar);

  return (
    <button onClick={() => themeVar(theme === "light" ? "dark" : "light")}>
      Toggle Theme (Current: {theme})
    </button>
  );
}
```
**Best for:** Managing local state **without Redux or React Context**.

---

## **🛠️ Summary of Apollo Hooks**
| Hook | Use Case |
|------|---------|
| `useQuery` | Fetches **data immediately** on mount |
| `useLazyQuery` | Fetches **data on demand** (button clicks, etc.) |
| `useMutation` | Executes **GraphQL mutations** (create, update, delete) |
| `useSubscription` | **Listens for real-time updates** |
| `useApolloClient` | **Direct access** to Apollo Client instance |
| `useFragment` | Reads **fragments** from the cache |
| `useReactiveVar` | Manages **local state with reactive variables** |

---

## **🚀 Which Hook Should You Use?**
- **Use `useQuery`** when you need **data to load immediately**.
- **Use `useLazyQuery`** when you want **to fetch data on demand**.
- **Use `useMutation`** when **sending data to the server**.
- **Use `useSubscription`** for **real-time data updates**.
- **Use `useApolloClient`** for **manual queries & cache management**.
- **Use `useReactiveVar`** for **managing local state without Redux**.

## Challenge! 
Follow the guide below. replace `useQuery` with `useLazyQuery` in your React Weather client. Notice that you will be rewriting your query to use variables. The variables are passed as an argument to the `executeQuery` function. 

### **`useLazyQuery` in Apollo Client**
The `useLazyQuery` hook in Apollo Client allows you to execute GraphQL queries *on demand*, rather than running them immediately when a component renders. This is useful when you need to fetch data in response to a user action, such as clicking a button or submitting a form.

---

### **🔹 Syntax:**
```jsx
const [executeQuery, { loading, error, data }] = useLazyQuery(QUERY, {
  variables: { someVariable: value },
});
```

---

### **🔹 Key Differences Between `useQuery` and `useLazyQuery`**
| Feature          | `useQuery` | `useLazyQuery` |
|-----------------|------------|----------------|
| **Execution Timing** | Runs immediately on component mount | Runs only when manually triggered |
| **Use Case** | Fetching data automatically | Fetching data based on user action |
| **Return Value** | `{ loading, error, data }` | `[executeQuery, { loading, error, data }]` |
| **Variables** | Passed once at mount, refetched on dependency change | Passed when `executeQuery` is called |

---

### **🔹 Example: Fetching Data on Button Click**
```jsx
import { gql, useLazyQuery } from '@apollo/client';
import { useState } from 'react';

const GET_WEATHER = gql`
  query GetWeather($zip: Int!) {
    getWeather(zip: $zip, units: imperial) {
      temperature
      description
    }
  }
`;

function WeatherComponent() {
  const [zip, setZip] = useState('');
  const [getWeather, { loading, error, data }] = useLazyQuery(GET_WEATHER);

  return (
    <div>
      <input 
        type="number"
        placeholder="Enter ZIP code"
        value={zip}
        onChange={(e) => setZip(e.target.value)}
      />
      <button onClick={() => getWeather({ variables: { zip: parseInt(zip) } })}>
        Get Weather
      </button>

      {loading && <p>Loading...</p>}
      {error && <p>Error: {error.message}</p>}
      {data && (
        <div>
          <p>Temperature: {data.getWeather.temperature}°F</p>
          <p>Description: {data.getWeather.description}</p>
        </div>
      )}
    </div>
  );
}

export default WeatherComponent;
```
---

### **🔹 How It Works**
1. `useLazyQuery(GET_WEATHER)` returns a function (`getWeather`) and an object containing `{ loading, error, data }`.
2. The query **does not run initially**.
3. When the button is clicked, `getWeather({ variables: { zip: parseInt(zip) } })` executes the query.
4. The component re-renders when `loading`, `error`, or `data` updates.

---

### **🔹 When to Use `useLazyQuery`**
✅ Fetching data **only when necessary** (e.g., search forms, filtering).  
✅ Avoiding unnecessary queries when **data isn’t always needed**.  
✅ Enhancing performance by preventing **unnecessary API calls on mount**.

---

### **🔹 Common Mistakes**
❌ **Forgetting to call the query function:**  
```js
const [getWeather, { data }] = useLazyQuery(GET_WEATHER);
console.log(data); // Will be undefined until `getWeather` is called!
```
✔ **Solution:** Call `getWeather()` when needed.

❌ **Not handling variables properly:**  
```js
getWeather({ variables: { zip } }); // If zip is a string, Apollo may throw an error!
```
✔ **Solution:** Ensure the correct type:
```js
getWeather({ variables: { zip: parseInt(zip, 10) } });
```

---

### **🔹 Summary**
- **`useLazyQuery` is best for on-demand queries.**
- **It does not run on mount; you must manually trigger it.**
- **Useful for forms, search bars, and conditional fetching.**

