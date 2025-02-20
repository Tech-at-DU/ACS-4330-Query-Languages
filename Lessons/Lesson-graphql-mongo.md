# Apollo Server + MongoDB

This starter code creates GraphQL server using Apollo. The server supports CRUD operations.

Get the project files here: https://github.com/Tech-at-DU/apollo-mongo-graphql-starter

The project uses `ts-node`, a TypeScript engine for Node.js. https://www.npmjs.com/package/ts-node

It will run using `nodemon` without having to compile: https://www.npmjs.com/package/nodemon

The Graphiql interface used by Apollo Server only seemed to work for me in Chrome! If you're having trouble, try Chrome. This seems to be a cors issue of some kind. 

# Project Files 

## `src/db.ts`
### **📌 `db.ts` – Database Connection Module**

This file is responsible for **connecting to the MongoDB database** using the **MongoDB Node.js driver**. It ensures that the connection is established once and reused across the application.

---

### **🛠️ Key Features of `db.ts`**
1. **Loads Environment Variables**  
   - Uses `dotenv.config()` to load the `MONGO_URI` from the `.env` file.  
   - Defaults to `"mongodb://127.0.0.1:27017/graphqlDB"` if no environment variable is set.

2. **Creates a Singleton Database Connection**  
   - Declares `client` (MongoDB client) and `db` (database instance) **outside the function** to maintain a **single connection**.
   - If a connection already exists, it **reuses** it instead of creating a new one.

3. **`connectToDB()` Function**
   - **Creates and connects** to a MongoDB instance using `MongoClient`.
   - **Enforces strict API rules** to catch deprecated methods early.
   - Prints `"✅ Connected to MongoDB"` when the connection is successful.

4. **Exports**
   - The `connectToDB()` function allows other modules to connect when needed.
   - `db` is exported directly for queries (but isn’t set until `connectToDB()` is called).


## `src/server.ts`
### **📌 `server.ts` – The Entry Point of the GraphQL Server**  

This file is the **main entry point** of the backend. It initializes and starts an **Apollo Server**, which handles GraphQL queries and mutations.  

---

### **🛠️ Key Features of `server.ts`**
1. **Loads Environment Variables**  
   - Uses `dotenv.config()` to load any environment variables from a `.env` file.

2. **Connects to MongoDB**  
   - Calls `connectToDB()` **before starting the server** to ensure that the database connection is ready.

3. **Creates an Apollo GraphQL Server**  
   - Uses the `ApolloServer` class from `apollo-server`.  
   - Loads **GraphQL schema (`typeDefs`)** and **resolvers (`resolvers`)** from the `graphql/` folder.

4. **Starts the Server**
   - Calls `server.listen()`, which:
     - Starts the server on a default port (**4000** unless otherwise specified).
     - Logs `"🚀 Server ready at http://localhost:4000/"` when successful.

## `src/graphql/schema.ts`
### **📌 `schema.ts` – GraphQL Schema Definition**  

This file defines the **GraphQL schema** for the application using `gql` from `apollo-server`. The schema specifies **what data can be queried and modified** in the API.

---

### **🛠️ Key Features of `schema.ts`**
1. **Defines the `Character` Type**  
   ```graphql
   type Character {
     id: ID!
     name: String!
     initiative: Int
   }
   ```
   - Each character has:
     - An **ID** (`id: ID!`) – Unique identifier.
     - A **name** (`name: String!`) – Required.
     - An **initiative score** (`initiative: Int`) – Optional.

2. **Queries (`Query` Type)**
   ```graphql
   type Query {
     characters: [Character!]!
   }
   ```
   - **`characters`**: Fetches an array of all characters.

3. **Mutations (`Mutation` Type)**
   ```graphql
   type Mutation {
     createCharacter(name: String!, initiative: Int): Character!
     updateCharacter(id: ID! name: String, initiative: Int): Character!
     deleteCharacter(id: ID!): Boolean
   }
   ```
   - **`createCharacter(name, initiative)`** → Creates a new character.
   - **`updateCharacter(id, name, initiative)`** → Updates an existing character.
   - **`deleteCharacter(id)`** → Deletes a character and returns `true` if successful.

---

### **🚀 How to Use It**
1. **Expand the API**  
   - Add new types (e.g., `Weapon`, `Spell`).  
   - Modify queries to filter or sort characters.  

2. **Test in GraphQL Playground**  
   ```graphql
   mutation {
     createCharacter(name: "Aragorn", initiative: 5) {
       id
       name
       initiative
     }
   }
   ```

## `src/graphql/resolvers.ts`
### **📌 `resolvers.ts` – GraphQL Resolvers**  

This file contains **the logic** behind each GraphQL query and mutation. It connects the GraphQL API to **MongoDB**, allowing the application to fetch, create, update, and delete data.

---

### **🛠️ Key Features of `resolvers.ts`**
1. **Handles Queries (`Query`)**
   ```ts
   characters: async () => { ... }
   ```
   - Fetches all characters from MongoDB.
   - Converts `_id` (MongoDB’s unique identifier) to a string for GraphQL compatibility.

2. **Handles Mutations (`Mutation`)**
   - **`createCharacter(name, initiative)`**
     - Inserts a new character into the database.
     - Returns the created character with its MongoDB-generated `id`.
   - **`updateCharacter(id, name, initiative)`**
     - Updates an existing character **only if the provided fields are non-null**.
     - Ensures the character exists before updating.
   - **`deleteCharacter(id)`**
     - Deletes a character by ID.
     - Returns `true` if deleted, `false` if no matching character was found.

## `src/models/Characters.ts`
### **📌 `Character.ts` – Mongoose Model for Characters**  

This file defines the **MongoDB schema** for storing characters using **Mongoose**, an Object Data Modeling (ODM) library for MongoDB.

---

### **🛠️ Key Features of `Character.ts`**
1. **Defines the Schema (`characterSchema`)**
   ```ts
   const characterSchema = new mongoose.Schema({
     name: { type: String, required: true },
     initiative: { type: Number, required: false },
   });
   ```
   - **`name`**: Required field (every character must have a name).
   - **`initiative`**: Optional number (default is `null` if not provided).

2. **Creates the Model (`Character`)**
   ```ts
   const Character = mongoose.model("Character", characterSchema);
   ```
   - Converts the schema into a **model**, allowing database operations like:
     ```ts
     Character.find();       // Get all characters
     Character.create({...}) // Add a new character
     Character.findById(id)  // Get a specific character
     ```

3. **Exports the Model**
   - `Character` is exported so that other files (like `resolvers.ts`) can interact with the database.

## Apollo Client cache

Read about the Apollo Client Cache here: https://www.apollographql.com/docs/react/caching/overview

### TL;DR: Apollo Client Cache
Apollo Client uses an **in-memory cache** to store GraphQL query results, reducing unnecessary network requests and improving performance.

---

### 🔹 Key Concepts
1. **Normalized Cache**  
   - Apollo stores data in a normalized format, similar to a database.  
   - Each object is stored by its `id`, preventing duplicate data.

2. **`cache.modify()` for Manual Updates**  
   - Used after mutations to **update specific fields** without refetching.  
   - Example: Updating a list of characters after one is edited.
   ```javascript
   cache.modify({
     fields: {
       characters(existingCharacters = [], { readField }) {
         return existingCharacters.map((characterRef) =>
           readField("id", characterRef) === updatedCharacter.id
             ? { ...characterRef, ...updatedCharacter }
             : characterRef
         );
       }
     }
   });
   ```

3. **Automatic Cache Updates**
   - If a query uses `id` and `__typename`, Apollo **automatically updates cached data** after a mutation.
   - Works when the mutation response contains the updated object.

4. **`refetchQueries` for Simplicity**
   - Forces a re-fetch of specific queries after a mutation.
   ```javascript
   const [updateCharacter] = useMutation(UPDATE_CHARACTER, {
     refetchQueries: ["GetCharacters"], // Name of query in cache
   });
   ```
   - **Use this if manual cache updates are too complex.**

5. **Evicting & Resetting Cache**
   - `cache.evict({ id: "Character:123" })` → Removes specific object.
   - `cache.reset()` → Clears everything (useful on logout).

---

### 🚀 TL;DR Summary
| Feature | What It Does |
|---------|-------------|
| **Normalized Cache** | Stores GraphQL data like a database. |
| **`cache.modify()`** | Manually updates cache after mutations. |
| **Auto Updates** | Works if `id` & `__typename` exist in response. |
| **`refetchQueries`** | Refetches specific queries instead of manual updates. |
| **`cache.evict()`** | Removes specific items from the cache. |

👉 **Use auto-updates when possible, `cache.modify()` for efficiency, and `refetchQueries` for simplicity!** 🚀🔥

### 🔍 Understanding `update()` in Apollo Client's `useMutation` Hook
The `update()` function in Apollo Client allows us to manually update the **Apollo cache** after a mutation runs. This prevents an unnecessary refetch from the server and ensures the UI reflects the most recent state.

---

### ✅ Breakdown of What’s Happening
```javascript
const [updateCharacter, { error: updateError }] = useMutation(UPDATE_CHARACTER, {
  update(cache, { data: { updateCharacter } }) {
    cache.modify({
      fields: {
        characters(existingCharacters = [], { readField }) {
          return existingCharacters.map((characterRef) =>
            readField("id", characterRef) === updateCharacter.id
              ? { ...characterRef, ...updateCharacter }
              : characterRef
          );
        }
      }
    });
  }
});
```

---

### 🚀 Step-by-Step Execution

#### 1️⃣ The Mutation Executes
- When `updateCharacter()` is called, it sends a mutation request to the GraphQL server.
- Once the server responds with updated data, **Apollo calls the `update()` function** to modify the cache.

#### 2️⃣ The `update()` Function Runs
```javascript
update(cache, { data: { updateCharacter } }) {
```
- The `cache` object represents Apollo’s in-memory cache.
- `{ data: { updateCharacter } }` extracts the updated character from the mutation response.

#### 3️⃣ `cache.modify()` Updates the Cache
```javascript
cache.modify({
  fields: {
    characters(existingCharacters = [], { readField }) {
```
- **`cache.modify()`** is a built-in Apollo function that allows us to update **specific fields** in the cache.
- **`characters`** is the name of the field in the Apollo cache storing the list of characters.
- **`existingCharacters`** represents the array of character references currently stored in Apollo's cache.

#### 4️⃣ Updating the Characters List
```javascript
return existingCharacters.map((characterRef) =>
  readField("id", characterRef) === updateCharacter.id
    ? { ...characterRef, ...updateCharacter }
    : characterRef
);
```
- **`readField("id", characterRef")`** extracts the `id` from each character stored in the cache.
- It **checks if the `id` matches** the one from `updateCharacter` (the updated data).
- **If it matches**, the character **is replaced with the updated data**.
- **If it doesn’t match**, the character remains unchanged.

---

### 🔥 Why Is This Important?
1. **Ensures UI Updates Immediately**  
   - Without this, the UI might **not reflect the update** until the next refetch.
   
2. **Prevents Unnecessary Network Requests**  
   - Since the cache is updated manually, **Apollo doesn’t need to fetch fresh data from the server**.

3. **Efficient & Optimized Performance**  
   - `cache.modify()` ensures that **only the affected character is updated**, instead of refetching the entire list.

---

### 🔧 Alternative: Using `refetchQueries` Instead
If cache modification is too complex, you can **force a refetch**:
```javascript
const [updateCharacter] = useMutation(UPDATE_CHARACTER, {
  refetchQueries: ["GetCharacters"], // Query name from Apollo's cache
});
```
This will **refetch the entire `characters` list** from the server instead of modifying the cache manually.

---

### 🎯 Summary
| Step | What Happens? |
|------|--------------|
| **1️⃣ Mutation Runs** | Sends `updateCharacter` mutation to the server. |
| **2️⃣ Server Responds** | Server returns updated character data. |
| **3️⃣ `update()` Runs** | Apollo calls `update()` to modify the cache. |
| **4️⃣ `cache.modify()` Updates Characters List** | Replaces the old character in the cache with the updated one. |
| **5️⃣ UI Updates Immediately** | React re-renders with the new character data. |

---

### 🚀 Final Thoughts
- This approach **manually updates the cache** instead of relying on a full refetch.
- It’s **efficient** but requires understanding Apollo’s cache structure.
- **Alternative:** Use `refetchQueries` if manual cache updates feel too complex.

Now, your app will **instantly update** the character list without an extra API request! 🚀🔥
