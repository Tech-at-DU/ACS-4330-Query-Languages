
## What is a websocket?
WebSockets are a protocol that enables persistent, bi-directional communication between a client (like a web browser) and a server over a single TCP connection. Here’s a detailed breakdown:

### Persistent Connection
Unlike traditional HTTP, which is request-response based, WebSockets establish a long-lived connection. After an initial handshake, the connection remains open, allowing data to be sent back and forth continuously without the overhead of repeatedly establishing new connections.

### Bi-directional Communication
With WebSockets, both the client and the server can initiate data transfers independently. This full-duplex communication means that the server can push data to the client as soon as it’s available—ideal for real-time applications like chat apps, live sports updates, or collaborative tools.

### How It Works
1. **Initial Handshake:**  
   The client initiates an HTTP request with an `Upgrade` header indicating it wants to switch protocols to WebSocket. The server, if it supports WebSockets, responds with a similar header, confirming the upgrade.

2. **Data Transfer:**  
   Once the handshake is complete, the connection switches to the WebSocket protocol. Data is exchanged in small packets called frames. These frames can carry text or binary data, making the exchange efficient.

3. **Connection Maintenance:**  
   The connection remains open until either the client or the server decides to close it, eliminating the need for repeated HTTP requests.

### Advantages Over HTTP
- **Reduced Latency:** Because the connection is maintained, messages can be sent immediately without waiting for a new connection to be set up.
- **Lower Overhead:** There’s no need to send HTTP headers with every message, making data transfer more efficient.
- **Real-Time Communication:** Ideal for applications that require instant updates, as data can be pushed in real-time from server to client and vice versa.

### Practical Applications
WebSockets are commonly used in:
- **Chat Applications:** Allowing users to send and receive messages in real time.
- **Live Notifications:** For updates such as news feeds or sports scores.
- **Online Gaming:** Enabling real-time interactions between players.
- **Collaborative Platforms:** Where multiple users interact with shared documents or projects simultaneously.

Overall, WebSockets provide a robust framework for real-time, low-latency communication in modern web applications, making them a popular choice for interactive, dynamic user experiences.

While WebSockets provide a powerful mechanism for real-time communication, they come with several drawbacks that can affect development and deployment:

- **Resource Consumption:**  
  Persistent connections require server resources to remain open for each client, which can be challenging to scale if thousands of clients are connected simultaneously.

- **Complexity in Infrastructure:**  
  Handling long-lived connections complicates aspects like load balancing and session management. Many standard HTTP tools (caches, proxies, etc.) aren’t designed for persistent connections, potentially leading to compatibility issues.

- **Security Concerns:**  
  WebSockets bypass some of the built-in security features of HTTP. This means developers must take extra precautions (such as using WSS for encryption) to prevent issues like cross-site WebSocket hijacking or unauthorized data access.

- **Debugging and Error Handling:**  
  Since WebSocket communication is asynchronous and stateful, diagnosing problems can be more challenging compared to traditional request-response models.

- **Fallback Requirements:**  
  Not all clients or network configurations support WebSockets natively. Implementing fallback mechanisms (like long polling) is often necessary to ensure broader compatibility.

Each of these factors means that while WebSockets are great for real-time apps, developers need to weigh these drawbacks and plan accordingly in terms of infrastructure, security, and maintenance.

# Lesson: Building a Real-Time WebSocket Chat App
Follow the instructions below to create your own websocket chat. Note that this is a simplified example and is missing many of the security and other features that would be required to make a robust profesional app. The code below is used to illustrate the basics of using websockets. 

## Overview
In this lesson, you will create a simple chat application using WebSockets. You will learn how to:
- Set up a Node.js project.
- Build a WebSocket server using the "ws" library.
- Create a basic HTML client with a chat UI.
- Implement real-time message broadcasting so that messages sent from one window appear in all connected windows.

## Objectives
By the end of this lesson, you will:
- Understand how WebSockets work for real-time, bidirectional communication.
- Set up and run a Node.js WebSocket server.
- Develop an HTML client that connects to the WebSocket server.
- Debug and test a multi-client chat application.

---

## Step 1: Setting Up the Project

1. **Create a New Directory**
  - Open your terminal or command prompt.
  - Create a new project directory:
    ```bash
    mkdir websocket-chat-app
    cd websocket-chat-app
    ```

2. **Initialize the Project**
  - Run the following command to create a `package.json` file:
    ```bash
    npm init -y
    ```

3. **Install the "ws" Library**
  - Install the WebSocket library by running:
    ```bash
    npm install ws
    ```

---

## Step 2: Creating the WebSocket Server

1. **Create a File Called `server.js`**
  - In your project directory, create a new file named `server.js`.

2. **Write the Server Code**
  - Copy and paste the following code into `server.js`:
    ```javascript
    // server.js
    const WebSocket = require('ws');

    // Create a new WebSocket server on port 8080
    const server = new WebSocket.Server({ port: 8080 });

    server.on('connection', socket => {
      console.log('Client connected');

      // Send a welcome message to the new client
      socket.send('Welcome to the real-time chat!');

      // When a message is received, broadcast it to all connected clients
      socket.on('message', message => {
        console.log(`Received: ${message}`);
        server.clients.forEach(client => {
          if (client.readyState === WebSocket.OPEN) {
            // Without .toString() message is a blob!
            client.send(message.toString());
          }
        });
      });

      // Log disconnections
      socket.on('close', () => {
        console.log('Client disconnected');
      });
    });

    console.log('WebSocket server is running on ws://localhost:8080');
    ```
  - **Explanation:**
    - **Connection Handling:** When a client connects, a welcome message is sent.
    - **Message Broadcasting:** Any message received is logged and then sent to every connected client.
    - **Logging:** Client connections and disconnections are logged in the console.

3. **Start the Server**
  - In your terminal, run:
    ```bash
    node server.js
    ```
  - You should see the message: "WebSocket server is running on ws://localhost:8080".

---

## Step 3: Creating the HTML Client

1. **Create a File Called `index.html`**
  - In your project directory, create a new file named `index.html`.

2. **Write the Client Code**
  - Copy and paste the following HTML code into `client.html`:
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <title>Real-Time Chat</title>
      <style>
        #chat {
          border: 1px solid #ccc;
          height: 300px;
          overflow-y: scroll;
          padding: 10px;
          margin-bottom: 10px;
        }
        #messageInput {
          width: 80%;
          padding: 5px;
        }
        #sendBtn {
          padding: 5px 10px;
        }
      </style>
    </head>
    <body>
      <h1>Real-Time Chat</h1>
      <div id="chat"></div>
      <input type="text" id="messageInput" placeholder="Type your message...">
      <button id="sendBtn">Send</button>

      <script>
        const chat = document.getElementById('chat');
        const messageInput = document.getElementById('messageInput');
        const sendBtn = document.getElementById('sendBtn');

        // Connect to the WebSocket server
        const socket = new WebSocket('ws://localhost:8080');

        // Log connection status
        socket.addEventListener('open', () => {
          console.log('Connected to the server');
        });

        // Display incoming messages
        socket.addEventListener('message', event => {
          console.log('Received message:', event.data);
          const messageElem = document.createElement('p');
          messageElem.textContent = event.data;
          chat.appendChild(messageElem);
          // Auto-scroll to the bottom of the chat area
          chat.scrollTop = chat.scrollHeight;
        });

        // Function to send a message
        function sendMessage() {
          const message = messageInput.value;
          if (message) {
            socket.send(message);
            messageInput.value = '';
          }
        }

        // Send message on button click or when Enter is pressed
        sendBtn.addEventListener('click', sendMessage);
        messageInput.addEventListener('keyup', event => {
          if (event.key === 'Enter') {
            sendMessage();
          }
        });
      </script>
    </body>
    </html>
    ```
  - **Explanation:**
    - **UI Elements:** The HTML creates a chat display area, an input field, and a send button.
    - **WebSocket Connection:** The client connects to the server at `ws://localhost:8080` and listens for incoming messages.
    - **Event Handling:** When a message is received, it is added to the chat window; when the send button is clicked or the Enter key is pressed, the message is sent to the server.

---

## Step 4: Testing the Chat App

1. **Serve the HTML File**
  - To avoid issues related to the `file://` protocol, serve the client file using LiveServer in VS Code, or use another localhost server.  

2. **Open Multiple Windows or Tabs**
  - Open `index.html` in two different browser windows or tabs (or even in two different browsers).
  - Each window should display the welcome message from the server.

3. **Test Real-Time Communication**
  - Type a message in one window and press Send.
  - The message should appear in both windows, demonstrating the server’s broadcast functionality.

4. **Debugging**
  - Open the browser’s developer console (usually F12 or right-click → Inspect) in both windows to monitor the connection status and message logs.
  - Check the Node.js terminal to verify that both clients are connected and that the server is logging the broadcast actions.

---

## Conclusion

Students have now built a simple, real-time chat application using WebSockets. This project demonstrates:
- How persistent WebSocket connections enable real-time, bidirectional communication.
- The process of setting up a Node.js WebSocket server.
- Creating a basic client interface to send and receive messages.
- Debugging common issues related to multi-client communication.

---

## Stretch Goals (Optional)
- **Username Integration:**  
  Modify the client to include a username input and prefix messages with the sender's name.
- **Styling Enhancements:**  
  Enhance the chat UI with additional CSS for a more polished appearance.
- **Message History:**  
  Implement a simple mechanism to store and display past messages when a new client connects.

# GraphQL Subscriptions 
GraphQL Subscriptions extend the GraphQL specification to enable real-time communication between the client and server. Instead of the client polling for updates, it can subscribe to specific events, and the server pushes new data as soon as it becomes available. This is typically implemented over WebSocket connections and is useful for applications that need live updates, such as chat apps, live feeds, or collaborative tools.

Below is an example of a simple GraphQL server that implements a real-time chat using subscriptions. In this example, we’ll use Apollo Server along with the built-in PubSub mechanism to broadcast new chat messages. When a client sends a message via a mutation, the server publishes the message to all subscribed clients.

### Step-by-Step Code Example (optional)

1. **Install Dependencies**

  First, create a new project directory, initialize npm, and install Apollo Server:

  ```bash
  mkdir graphql-chat
  cd graphql-chat
  npm init -y
  npm install apollo-server@^3.13.0 graphql@^16.10.0 graphql-subscriptions@^2.0.0 graphql-ws@^5.10.1 ws@^8.18.1
  ```

  The versions of these packages seem to matter! Some of the packages are deprecated. It seems like the intent is to use a different, more complicated, setup for subscriptions. I'm using the packages here to keep this example simple! 

2. **Create the Server Code**

  Create a file named `server.js` and paste the following code:

  ```javascript
  const { ApolloServer, gql } = require('apollo-server-express');
  const express = require('express');
  const { createServer } = require('http');
  const { WebSocketServer } = require('ws');
  // const { useServer } = require('graphql-ws/dist/use/ws.cjs'); 
  const { useServer } = require('graphql-ws/lib/use/ws')
  const { makeExecutableSchema } = require('@graphql-tools/schema');
  const { PubSub } = require('graphql-subscriptions');

  const typeDefs = gql`
    type Message {
      id: ID!
      content: String!
    }

    type Query {
      messages: [Message!]!
    }

    type Mutation {
      postMessage(content: String!): Message!
    }

    type Subscription {
      messageAdded: Message!
    }
  `;

  let messages = [];
  let idCounter = 1;

  const pubsub = new PubSub();

  const resolvers = {
    Query: {
      messages: () => messages,
    },
    Mutation: {
      postMessage: (_, { content }) => {
        const message = { id: idCounter++, content };
        messages.push(message);
        pubsub.publish('MESSAGE_ADDED', { messageAdded: message });
        return message;
      },
    },
    Subscription: {
      messageAdded: {
        subscribe: (_, __, { pubsub }) => pubsub.asyncIterator('MESSAGE_ADDED'),
      },
    },
  };

  const schema = makeExecutableSchema({ typeDefs, resolvers });
  const app = express();
  const httpServer = createServer(app);

  const server = new ApolloServer({
    schema,
  });
  (async () => {
    await server.start();
    server.applyMiddleware({ app });
    
    // Create a WebSocket server on the same HTTP server and at the same path
    const wsServer = new WebSocketServer({
      server: httpServer,
      path: server.graphqlPath,
    });

    // Attach the graphql-ws server using the proper API
    useServer(
      {
        schema,
        context: () => ({ pubsub }), // This makes pubsub available in subscription resolvers
      },
      wsServer
    );

    const PORT = 4000;
    httpServer.listen(PORT, () => {
      console.log(`Server is now running on http://localhost:${PORT}${server.graphqlPath}`);
    });
  })();
  ```

### How It Works

- **Schema Definition:**
  - The `Message` type defines a chat message.
  - The `Query` type allows fetching all messages.
  - The `Mutation` type provides a `postMessage` operation that adds a new message.
  - The `Subscription` type defines `messageAdded` which notifies subscribers of new messages.

- **Resolvers:**
  - The `postMessage` mutation creates a new message, adds it to an in-memory list, and uses the PubSub system to publish a `MESSAGE_ADDED` event.
  - The `messageAdded` subscription listens for the `MESSAGE_ADDED` event and pushes new messages to any connected client.

- **Running the Server:**
  - When you run `node server.js`, the server will be available at the provided URL.
  - The subscription endpoint is also provided so you can use tools like Apollo Studio or GraphQL Playground to test real-time updates.

### Testing the Chat App

1. **Run the Server:**

  ```bash
  node server.js
  ```

2. **Using GraphQL Playground (or Apollo Studio Explorer):**

  - **To Post a Message:**  
    Use the following mutation in the playground:

    ```graphql
    mutation {
      postMessage(content: "Hello, GraphQL Subscriptions!") {
        id
        content
      }
    }
    ```

   - **To Subscribe to New Messages:**  
    Open a new tab in the playground and run:

    ```graphql
    subscription {
      messageAdded {
        id
        content
      }
    }
    ```

  When you execute the mutation, any client subscribed to `messageAdded` will receive the new message in real time.

