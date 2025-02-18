---
title: "WebSockets in FastAPI: From Basics to Scaling"
date: 2025-02-15
summary: "WebSockets enable real-time, bidirectional communication, perfect for chat apps, live notifications, and gaming. FastAPI, a modern Python framework, makes it easy to implement WebSockets for scalable, real-time applications. This blog walks you through setting up WebSocket connections, handling messages, and managing client-server interactions in FastAPI, with practical examples and best practices."
description: "Implement WebSockets in FastAPI, from basic setups for small projects to advanced, scalable solutions for industry-level applications, with practical examples and best practices."
toc: false
readTime: true
autonumber: false
math: false
tags: ["FastAPI", "Websocket", "Communication", "Realtime", "Redis", "Python", "Postgresql"]
showTags: true
hideBackToTop: false
categories: ["Engineering"]
weight: 1
draft: false
---

## Websocket and Why You Should Care?

Let’s talk about WebSockets. If HTTP is like sending letters (write, wait, receive, repeat), WebSockets are like a never-ending phone call. They allow real-time, bidirectional communication between a client and a server, meaning both can send and receive data instantly without waiting for requests.

Technically, WebSockets are a protocol that provides full-duplex communication over a single, long-lived connection. Unlike HTTP, which is stateless and requires a new request for every response, WebSockets keep the connection alive. This makes them perfect for:

- Chat apps (because no one likes waiting for “…” to turn into a message).
- Live notifications (because FOMO is real).
- Real-time data sharing (like the app we’re building in this blog).

I’m building a real-time location-sharing app for my motorcycle riding group. One of the key features is sharing live locations (with consent, of course). No more lying to the admins when you say, “I’m about to reach,” while you’re still stuck in Silk Board traffic. (Pro tip: You should’ve started early.)

To make this happen, I’m using FastAPI (Python) for the backend because it’s fast, async, and just plain awesome. For handling moderate scale, I’m using Redis to keep things snappy with its in-memory data storage. And for large-scale scenarios (fingers crossed, I hope my app gets so popular that I’ll have this “good problem” to solve), I’ll bring in RabbitMQ for message brokering. It’s my way of future-proofing the app, just in case.

In this blog, we’ll go from “Hey, I connected two things!” to “OMG, I’m handling thousands of users without breaking a sweat!” So, grab a coffee (or a beer, no one’s judging), and let’s get started.


## The Setup

Before we dive into the code, let’s get our tools ready. We’ll need FastAPI for the backend, Redis for handling real-time data, and Pika for working with RabbitMQ later.

#### Step 1: Install FastAPI

FastAPI is the backbone of our app. It’s fast, async, and ridiculously easy to use. Let’s install it along with uvicorn and websockets, an ASGI server to run our app:

```shell
pip install fastapi uvicorn websockets
```

#### Step 1: Install Redis

Redis is perfect for real-time data handling. It’s fast, in-memory, and works like a charm for our use case. Let’s install the Redis Python client:

```shell
pip install redis
```

*Pro tip: If you don’t have Redis installed on your system, just search for <span class="span_highlight"> How to set up Redis on [your OS]</span> (Linux, Mac, Windows, etc.). It’s straightforward, and there are plenty of guides out there!*

#### Step 3: Install Pika for RabbitMQ

For scaling to thousands of users, we’ll use RabbitMQ. To interact with it in Python, we’ll use Pika:

```shell
pip install pika
```

#### Why This Setup?
- <span class="span_highlight">FastAPI:</span> Because it’s fast, modern, and perfect for real-time apps.
- <span class="span_highlight">Redis:</span> For in-memory data storage and real-time updates.
- <span class="span_highlight">Pika:</span> To handle message brokering when we scale (because dreams do come true, right?).


## Writing the Code: Let’s Get Real-Time

Let’s start by writing some code! We’ll set up a basic FastAPI app, serve an HTML page on the root route, and run a WebSocket server.

#### Step 1: Create the FastAPI App

```python
# main.py

from fastapi import FastAPI, WebSocket
from fastapi.responses import HTMLResponse

app = FastAPI()

# Serve an HTML page on the root route
@app.get("/")
async def get():
    return HTMLResponse("""
    <html>
        <head>
            <title>Real-Time Data Sharing</title>
        </head>
        <body>
            <h1>Welcome to the Real-Time App!</h1>
            <div id="chat">
                <input type="text" id="message" placeholder="Type a message...">
                <button onclick="sendMessage()">Send</button>
            </div>
            <ul id="messages"></ul>

            <script>
                // Generate a random username (e.g., "AnonymousXYZ")
                const generateUsername = () => {
                    const randomPart = Math.random().toString(36).substring(2, 5).toUpperCase();
                    return `Anonymous${randomPart}`;
                };

                const username = generateUsername(); // Assign a random username
                const ws = new WebSocket("ws://localhost:8081/ws");

                // Function to send a message
                const sendMessage = () => {
                    const message = document.getElementById("message").value.trim();
                    if (message === "") return; // Prevent sending empty messages

                    ws.send(`${username}: ${message}`);
                    document.getElementById("message").value = ""; // Clear the input box

                };

                // Display incoming messages
                ws.onmessage = (event) => {
                    const messages = document.getElementById("messages");
                    const li = document.createElement("li");
                    li.textContent = event.data;
                    messages.appendChild(li);
                };
            </script>
        </body>
    </html>
""")
# Run the app
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8080)

```

```python
# web_socket.py

from fastapi import FastAPI, WebSocket
from fastapi.responses import HTMLResponse

app = FastAPI()

# WebSocket endpoint
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    print("INFO: WebSocket connection established")  # Log connection
    while True:
        data = await websocket.receive_text()
        print(f"INFO: Received: {data}")  # Log received message
        await websocket.send_text(f"{data}")  # Broadcast the message
        print(f"INFO: Sent: {data}")  # Log sent message

```



#### Step 2: Run the WebSocket Server

Since WebSockets need to run indefinitely, we’ll run the WebSocket server on a different port (8081). This avoids potential conflicts and makes debugging easier.

##### Running the HTTP Server (Port 8080)
The HTTP server will serve the HTML page at the root route. Open a terminal and run:

```shell
uvicorn main:app --host 0.0.0.0 --port 8080
```

##### Running the WebSocket Server (Port 8081)
The WebSocket server will handle real-time communication. Open another terminal and run:

```shell
uvicorn web_socket:app --host 0.0.0.0 --port 8081
```


##### Why Different Ports?
We’re running the HTTP server on port 8080 and the WebSocket server on port 8081 because:
- <span class="span_highlight">Separation of Concerns:</span> Keeps HTTP and WebSocket traffic isolated.
- <span class="span_highlight">Easier Debugging:</span> If something goes wrong, you know exactly where to look.
- <span class="span_highlight">Scalability:</span> Makes it easier to scale and load balance in the future.


And hey, if you’re curious about the pros and cons of running them on the same port, stay tuned for a future blog post (because who doesn’t love a good sequel?).


#### Step 3: Verify Everything is Working

Let’s make sure everything is set up correctly.

- Open <span class="span_highlight">[http://localhost:8080](http://localhost:8080)</span> in your browser. You should see the chat interface.
- Check the Network Tab in developer tools <span class="span_highlight">(F12)</span>. Look for a WebSocket connection to <span class="span_highlight">[ws://localhost:8081/ws](ws://localhost:8081/ws)</span> with status 101 Switching Protocols.
- Type you message and submit, In the terminal running the WebSocket server, you should see logs like:

```plaintext
INFO: WebSocket connection established
INFO: Received: AnonymousABC: Hello!
INFO: Sent: AnonymousABC: Hello!
```

#### Step 4: Making the Chat Bidirectional (Because Talking to Yourself is Weird)

Right now, our chat app is like a monologue—you send a message, and it echoes back to you. But let’s be honest, talking to yourself is only fun for so long. To fix this, we’ll:


- <span class="span_highlight">Track all connected users:</span> So the server knows who’s chatting.
- <span class="span_highlight">Broadcast messages:</span> So everyone in the group can see them (because group chats are where the drama happens).

##### Create a Connection Manager
This guy will keep track of who’s connected and broadcast messages like a town crier.

```Python
# connection_manager.py

from fastapi import WebSocket
from typing import List

class ConnectionManager:
    def __init__(self):
        self.active_connections: List[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)
        print(f"INFO: New connection. Total connections: {len(self.active_connections)}")

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)
        print(f"INFO: Connection removed. Total connections: {len(self.active_connections)}")

    async def broadcast(self, message: str):
        for connection in self.active_connections:
            await connection.send_text(message)
        print(f"INFO: Broadcasted: {message}")

```

##### Update the WebSocket Endpoint
Now, the server will broadcast messages to everyone, not just the sender.

```python
# web_socket.py

from fastapi import FastAPI, WebSocket
from connection_manager import ConnectionManager

app = FastAPI()

manager = ConnectionManager()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await manager.connect(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            print(f"INFO: Received: {data}")
            await manager.broadcast(data)  # Send to everyone
    except Exception as e:
        print(f"ERROR: {e}")
    finally:
        manager.disconnect(websocket)

```

Now your chat app is fully functional and ready for some real-time drama. Next up: scaling it like a pro. Stay tuned!


#### Step 5: How Far Can We Go with this setup?

Our current setup uses a simple `ConnectionManager` to handle WebSocket connections and broadcast messages. But how many users can it handle before things start falling apart? Let’s do some math and find out.

##### Back-of-the-Envelope Calculations

<b>Memory Usage</b>

- Each WebSocket connection consumes ~10 KB of memory (this can vary based on the server and framework).
- If your server has 8 GB of RAM available for the app:

```plaintext
Total connections = Total RAM / Memory per connection
Total connections = 8 GB / 10 KB = 800,000 connections
```
But wait! This is just memory. In reality, CPU and network limits will kick in much earlier.

<b>CPU Usage</b>

- Broadcasting a message to N users requires O(N) CPU operations.
- If you have 1,000 users and each sends 10 messages per second:

```plaintext
Messages per second = 1,000 users * 10 messages = 10,000 messages/second
```

<b>Network Bandwidth</b>
If each message is 100 bytes, and you have 1,000 users sending 10 messages per second:

```plaintext
Bandwidth = 1,000 users * 10 messages * 100 bytes = 1 MB/s
```
This is manageable for now, but as users grow, bandwidth becomes a bottleneck.

<b>Practical Limits</b>
- <span class="span_highlight">Single Server:</span>  With 4 CPU cores and 8 GB RAM, you can handle ~1,000–2,000 concurrent users before performance degrades.
- <span class="span_highlight">Bottlenecks:</span>
  - CPU struggles with broadcasting.
  - Memory fills up as connections grow.
  - Network bandwidth becomes a limiting factor.

    
<b>What Happens When You Hit the Limit?</b>
- <span class="span_highlight">Latency:</span> Messages take longer to deliver.
- <span class="span_highlight">Timeouts:</span> New connections may fail.
- <span class="span_highlight">Crashes:</span> The server might run out of memory or CPU.

<b>What If You’re Running the HTTP Server on the Same Machine?</b>

- The HTTP server will share the same CPU, memory, and network resources.
- This reduces the available resources for WebSocket connections, lowering the practical limit to ~500–1,000 concurrent users.


Without these, you’re stuck with the limits of a single server. In the next section, we’ll explore scaling with Redis. Stay tuned!

## Scaling Up: Handling Moderate Traffic
Our current setup can handle ~1,000–2,000 users, but what if your app goes viral? (Hey, we can dream, right?) Let’s scale things up with Redis.

#### Why Redis?
Redis is an in-memory data store that’s perfect for:

- Storing WebSocket connections: Track users across multiple servers.
- Pub/Sub Messaging: Broadcast messages efficiently.
- Scalability: Handle thousands of connections without breaking a sweat.

#### Let's Add Redis Connection Manager

We’ll create a singleton Redis connection to manage WebSocket connections and messages.


```python
# reddis_connection_manager.py

import redis
import json
import asyncio

class RedisManager:
    def __init__(self):
        self.redis = redis.Redis(host="localhost", port=6379, db=0)
        self.pubsub = self.redis.pubsub()
        self.active_connections = set()

    async def subscribe(self, websocket, channel):
        self.active_connections.add(websocket)
        self.pubsub.subscribe(channel)
        await self.listen(websocket, channel)

    async def listen(self, websocket, channel):
        for message in self.pubsub.listen():
            if message["type"] == "message":
                data = json.loads(message["data"])
                await websocket.send_text(json.dumps(data))  # Send to the client

    def publish(self, channel, message):
        self.redis.publish(channel, json.dumps(message))

    async def unsubscribe(self, websocket, channel):
        self.active_connections.remove(websocket)
        self.pubsub.unsubscribe(channel)

```

#### Update the WebSocket Endpoint

```python
# web_socket.py
from fastapi import FastAPI, WebSocket
from reddis_connection_manager import RedisManager

app = FastAPI()

redis_manager = RedisManager()


@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    redis_manager.subscribe("chat")  # Subscribe to the "chat" channel
    username = None
    try:
      # Receive the username from the frontend
        username = await websocket.receive_text()
        print(f"INFO: {username} connected")
        while True:
            data = await websocket.receive_text()
            redis_manager.publish("chat", {"user": username, "message": data})  # Broadcast to Redis
    except Exception as e:
        print(f"ERROR: {e}")
    finally:
        redis_manager.pubsub.unsubscribe("chat")
        print(f"INFO: {username} disconnected")

```
Use this in the html response for the root page

```javascript

  // Generate a random username (e.g., "AnonymousXYZ")
  const generateUsername = () => {
      const randomPart = Math.random().toString(36).substring(2, 5).toUpperCase();
      return `Anonymous${randomPart}`;
  };

  const username = generateUsername(); // Assign a random username
  const ws = new WebSocket("ws://localhost:8081/ws");

  // Send the username to the server
  ws.onopen = () => {
      ws.send(username); // Send the username first
  };

  // Function to send a message
  const sendMessage = () => {
      const message = document.getElementById("message").value;
      if (message.trim() === "") return; // Don't send empty messages
      ws.send(message); // Send only the message (username is already known)
      document.getElementById("message").value = ""; // Clear the input box
  };

  // Display incoming messages
  ws.onmessage = (event) => {
      const data = JSON.parse(event.data); // Parse the message
      const messages = document.getElementById("messages");
      const li = document.createElement("li");
      li.textContent = `${data.user}: ${data.message}`;
      messages.appendChild(li);
  };
    
```

####  Scaling with Redis (Math Edition)

Let’s compare our Python ConnectionManager with Redis and see how Redis saves the day.

<b>Python ConnectionManager</b>
- <span class="span_highlight">Memory:</span> Each WebSocket connection consumes ~10 KB. 8 GB RAM → ~800,000 connections (theoretically).
- <span class="span_highlight">CPU:</span> Broadcasting to N users requires O(N) operations. 1,000 users sending 10 messages/second = 10,000 messages/second (CPU cries).
- <span class="span_highlight">Practical Limit: </span> ~1,000–2,000 users (CPU and memory choke), due to event loop limitations and GIL.

<b>Redis to the Rescue</b>

- <span class="span_highlight">Memory:</span> Redis stores only metadata and Pub/Sub references, reducing its per-connection memory footprint. 
8 GB → 800,000 connections assumption is rough but fair for estimation. 
However, in practice, network buffers and additional metadata would slightly reduce this number.
- <span class="span_highlight">CPU:</span> Redis Pub/Sub is not O(N) but significantly optimised. Internally, Redis Pub/Sub still
needs to push messages to each subscriber. However, redis minimizes context switching, optimizes memory access, and efficiently handles large numbers of subscribers compared to a Python-based event loop.
- <span class="span_highlight">Practical Limit:</span> ~10,000–20,000 concurrent users, (Redis flexes its muscles) but 
this depends on message frequency and server configuration. 
Some deployments (with Redis Cluster, Sentinel, or sharding) scale far beyond this. 


<b>Why Redis Wins</b>

- <span class="span_highlight">Optimized Broadcasting:</span>Pub/Sub efficiently delivers messages with minimal latency.
- <span class="span_highlight">Scalable Architecture:</span>Supports clustering and sharding for handling massive user loads.
- <span class="span_highlight">Lower CPU Overhead:</span>Offloads message distribution from the application server.
- <span class="span_highlight">High Throughput:</span>Handles thousands of messages per second with minimal performance loss.
- <span class="span_highlight">Reliable Scaling:</span>Works with load balancers and multiple Redis instances for redundancy.



## The Dream Scale: Handling Thousands of Users


## Conclusion: Real-Time Apps Made Simple


With Redis, you can handle 10x more users than with a Python ConnectionManager. And if your app goes viral, Redis will be your best friend (until you need RabbitMQ). Stay tuned for the next section, where we’ll scale even further!