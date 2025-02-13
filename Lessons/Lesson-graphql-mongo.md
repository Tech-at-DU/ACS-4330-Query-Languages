# Starter Code: Apollo Server + MongoDB
This starter code sets up a **GraphQL server** using **Apollo Server** 
and connects it to **MongoDB**. The server allows clients to query a collection 
of "spaces" from the database. Let’s break it down step by step.

"Spaces" in this example is a list of public spaces, it includes fields for title, 
description, hours, address, etc. You will be creating your own collections and 
defining your own fields on those collections.

**Download the project starter code here:** https://github.com/Tech-at-DU/apollo-mongo-graphql-starter

---

## 1. Project Setup
Before running this code, you need to:
- Install dependencies:  
  ```sh
  npm install @apollo/server @apollo/server/standalone mongodb dotenv
  ```
- Create a `.env` file to store database credentials:
  ```
  MONGO_USER=yourUsername
  MONGO_PASSWORD=yourPassword
  ```

---

## 2. Loading Environment Variables (`dotenv`)
```ts
import dotenv from 'dotenv';
dotenv.config();
```
- Loads environment variables from a `.env` file.
- These variables are used to store **sensitive information** (like database credentials) securely.

---

## 3. Setting Up MongoDB Connection
```ts
import { MongoClient, ServerApiVersion, Db } from 'mongodb';
```
- The `MongoClient` is used to connect to a MongoDB database.
- `Db` represents the database we will interact with.

### **🔹 Creating the Connection**
```ts
const mongoUser = process.env.MONGO_USER;
const mongoPassword = process.env.MONGO_PASSWORD;

if (!mongoUser || !mongoPassword) {
  throw new Error("Missing MongoDB credentials in environment variables.");
}

const uri = `mongodb+srv://${mongoUser}:${mongoPassword}@cluster0.q7yehdv.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0`;
```
- **Constructs the MongoDB connection string** using credentials stored in `.env`.
- **Throws an error if credentials are missing**, preventing accidental misconfiguration.

### 🔹 Connecting to MongoDB (Singleton Pattern)
```ts
let client: MongoClient | null = null;
let db: Db;

async function connectToDB() {
  if (!client) {
    client = new MongoClient(uri, {
      serverApi: {
        version: ServerApiVersion.v1,
        strict: true,
        deprecationErrors: true,
      },
    });

    await client.connect();
    db = client.db("test");
    console.log("Connected to MongoDB");
  }
  return db;
}
```
- **Ensures only one database connection is created (Singleton Pattern).**  
- The function `connectToDB()` returns the database instance.

---

## 4. Fetching Data from MongoDB
```ts
async function getSpaces() {
  try {
    const database = await connectToDB();
    return await database.collection("datas").find().toArray();
  } catch (error) {
    console.error("Error fetching spaces:", error);
    return [];
  }
}
```
- Connects to MongoDB and retrieves all documents from the `"datas"` collection.
- **Returns an array of spaces** or an empty array if an error occurs.

---

## 5. Defining the GraphQL Schema
```ts
const typeDefs = `
  type Geo {
    lat: Float
    lon: Float
  }

  type Space {
    _id: ID!  
    title: String
    desc: String
    address: String
    hours: String
    geo: Geo
    images: [String]
    features: [String]
  }

  type Query {
    spaces: [Space!]!
  }
`;
```
- Defines a **GraphQL schema** that describes the data structure:
  - `Space` represents a physical location.
  - `Geo` stores latitude and longitude.
  - `Query.spaces` fetches a list of spaces.

---

## 6. Implementing the Resolvers
```ts
const resolvers = {
  Query: {
    spaces: async () => {
      try {
        return await getSpaces();
      } catch (error) {
        console.error("GraphQL Resolver Error:", error);
        throw new Error("Failed to fetch spaces.");
      }
    },
  },
};
```
- A **resolver** is a function that defines how GraphQL queries retrieve data.
- The `spaces` query **calls `getSpaces()` to fetch data from MongoDB**.

---

## 7. Setting Up and Running the GraphQL Server
```ts
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';

const server = new ApolloServer({
  typeDefs,
  resolvers,
});

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 },
});
console.log(`🚀 Server ready at: ${url}`);
```
- **Creates an Apollo Server instance** with our schema and resolvers.
- **Starts the server on port `4000`**.

---

## 8. Graceful Shutdown (`SIGINT`)
```ts
process.on("SIGINT", async () => {
  if (client) {
    await client.close();
    console.log("MongoDB connection closed.");
    process.exit(0);
  }
});
```
- Ensures the database connection **closes properly when the server stops**.

---

## ✨ Next Steps for Students
Students can modify and expand the project by:
1. **Adding New Collections** – Modify `connectToDB()` to connect to different MongoDB collections.
2. **Creating New Queries** – Expand `typeDefs` and `resolvers` to support additional queries (e.g., fetching a single space by ID).
3. **Adding Mutations** – Implement `Mutation` to allow inserting or updating data.
4. **Deploying the Server** – Host the server on **Render, Railway, or Vercel**.

---

### 🚀 Summary
- **MongoDB stores "spaces" a collection describing public spaces in San Francisco.**
- **Apollo Server provides a GraphQL API to retrieve data.**
- **You can easily modify the schema and resolvers to build your own project.**