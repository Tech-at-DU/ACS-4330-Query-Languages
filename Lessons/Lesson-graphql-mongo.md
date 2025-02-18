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
