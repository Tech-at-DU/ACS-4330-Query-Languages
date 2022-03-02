# FEW 2.9 Final Assessment

Your goal is to build a GraphQL Todo application. For this assignment, you will build a server that supports your GraphQL schema. You will write the schema, resolvers, and some queries that test query types in your schema. 

# Todo GraphQL

Your goal is to make a GraphQL todo app. It should be able to:

- Display a list of todos
- Create new todos
- Mark a todo completed or not completed

You can store your list of todos in memory.

## Challenges

### Create a Server

- Setup a GraphQL server. Use your server of choice e.g. Express GraphQL
- Enable Graphiql

### Create a Schema 

Write a schema that defines the following types with fields listed:

**Todo type**
- name 
- completed
- date 
- id

**Query type**
- getAllTodos, should return a list of todos
- getTodo, should return a single todo
- getCompletedTodos, returns a list of completed todos
	- Stretch: can return completed or not completed todos

**Mutation type**
- addTodo, creates a new todo, and returns a Todo
- completeTodo, marks a todo complete and returns a Todo

### Write a GraphQL Queries

Using the **GraphQL query language** write queries to perform the following operations. 

**Test these in Graphiql and paste them into a readme in your project folder.** If I launch your project I should be able to test all of your queries in Graphiql when I test your project. 

- List all todos
- Add a new todo: name: "Complete the final assessment"
- Show the: "Completed final assessment" todo 
- Complete the: "Complete final assessment" todo
- Show all completed todos
- Show all not completed todos

### Stretch Challenge: Use Enum

Add an enum to prioritize your todos. 

**Schema**

- Define an enum with: High, Normal, and Low 
- Add a priority field to your Todo type
- Add a setPriority mutation
- Add a query to sort by priority

**Resolver**

Handle the new query types in your schema. You'll need to also update your data source. 

**Query**

Test the following queries and add them to your readme:  

- Display the priority when you list todos
- Set the "Complete final assessment" todo to "High" priority
- Sort todos by priority

### Stretch Challenge: Add Date type

Add a Date type. The date type should store a date in UTC. 

Add a Date field to the Todo type. 

## Submit your work 

Submit your completed work on GradeScope.

<!-- 

### Define a Schema

Enum Race 
- Human
- Dwarf
- Elf

Type Character
- name
- race
- power
- speed
- hp

Type Party 
- name 
- characters []

Query 
- getParty returns Party
- 

Mutation 
- createParty 
	- parameters name
	- returns Party
- creatCharacter returns Character
- addToParty return character

### Create a server 

- Setup GraphQL server 
- Define a resolver for your schema

### Write Queries 

Define the following queries

- get a party 
	- show 
 -->
