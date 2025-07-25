---
title: "Securing the websocket connection"
date: 2025-02-27
summary: "This blog extends our previous discussion on WebSockets with FastAPI. Sure, everything worked fine, but I conveniently skipped the most important part—security. Oops! Consider this my redemption post, where I’ll dive deep into securing WebSocket connections and, of course, sneak in some crude humor because why should security be boring?"
description: "Learn how to secure WebSocket connections by comparing session-based authentication and JWTs, understanding the importance of WSS over WS, and implementing additional security measures like message validation, rate limiting, and CORS policies. This blog builds on our FastAPI WebSocket implementation to ensure your real-time applications are both performant and secure."
toc: true
readTime: true
autonumber: false
math: false
tags: ["FastAPI", "Python", "Websocket", "Security", "JWT", "Session"]
showTags: true
hideBackToTop: false
categories: ["Engineering"]
weight: 1
draft: false
---

So, you've built the websocket connection using FastAPI, scaled using `Redis` or `RabbitMQ` and everything works like charm.
But here's one thing: If you are not securing your websocket, you're basically leaving the front door open with sign that says,
_Hackers Welcome Here!_.

This could lead to all sorts of problems for you and your users—problems you definitely don’t want. As an engineer, it’s your responsibility to build applications that are not just functional but also secure.

In this post, we’ll dive into all these problems and their solutions. So, grab your coffee, fire up your IDE, and let’s get started.


## The Setup 

In the last blog post, we built a WebSocket connection using FastAPI and even scaled it up using `Redis` and `RabbitMQ`. 
If you haven’t read that yet, go ahead and check it out here:
<a href="https://rjsnh1522.github.io/engineering/websocket_in_fast_api/" class="span_highlight" target="_blank">WebSockets in FastAPI: From Basics to Scaling</a>. Trust me, it’s worth it—unless you enjoy reading about security without any context, in which case, you do you.

We’ll be using the same codebase from that post and modifying it as needed. Why? Because rewriting everything from scratch is overrated, `CTRL+C` `CTRL+V` and modification after that is the way.
Plus, it’s always fun to see how a simple implementation evolves into something more robust (and secure).


## Why It Matters

Let’s take a real-life example: you’re building a chat application—maybe a one-to-one chat or a group chat. 
To make things interesting, let’s use the same famous names we’ve all come to know and love: Alice, Bob, Eve, and Mallory. 
(If you’re wondering why these names are so popular in cybersecurity, 
check out this fun read: <a href="https://bluegoatcyber.com/blog/who-are-alice-and-bob-in-cybersecurity/" class="span_highlight" target="_blank">Alice, Bob, Eve who?</a> 
But for now, let’s focus on securing WebSockets.)

Here's the scenario:

- <b>Alice</b> and <b>Bob</b> are happily chatting away, sharing their weekend plans and maybe even some sensitive info (like Bob’s embarrassing cat photos).
- <b>Eve</b>, the eavesdropper, is lurking on the same network. If your WebSocket connection isn’t secure (i.e., you’re using WS instead of WSS), Eve can easily intercept their messages. Suddenly, Bob’s cat photos are the least of your worries—Eve now has access to everything Alice and Bob are saying.
- <b>Mallory</b>, the malicious attacker, takes things a step further. She doesn’t just eavesdrop; she injects malicious code into the WebSocket connection, potentially crashing your server or stealing user data.

Now, imagine this happening in your app. Your users’ trust is shattered, your app’s reputation is ruined, and your app makes headlines for all the wrong reasons. And guess what? You’re left dealing with the fallout.

This isn’t just a hypothetical scenario—it’s happened many times in the past. Big and small organizations alike have lost customer data, faced lawsuits, and suffered irreversible damage to their reputation. You can read about some of the most infamous incidents here: <a href="https://en.wikipedia.org/wiki/List_of_data_breaches" target="_blank" class="span_highlight">List of Data Breaches</a>.

To make matters worse, governments around the world have introduced strict policies to hold companies accountable for data breaches. For example:

- <b class="span_highlight">GDPR (General Data Protection Regulation)</b> in the EU imposes fines of up to €20 million or 4% of global annual turnover (whichever is higher) for mishandling user data.
- <b class="span_highlight">CCPA (California Consumer Privacy Act)</b> in the US gives consumers the right to sue companies for data breaches, with penalties ranging from $100 to $750 per consumer per incident.
- <b class="span_highlight">PDPA (Personal Data Protection Act)</b> in Singapore and similar laws in other countries also enforce hefty fines for data breaches.

So, why take the risk? It’s better to make one less insecure thing in the world, starting with your WebSocket connections.

Let's take a look on all the options we have for securing the websocket connections.

## Session vs JWT: The Battle of Authentication
When it comes to securing WebSocket connections, authentication is like the bouncer at a club—it decides who gets in and who gets kicked out. But not all bouncers are created equal. Some are chill and easygoing (looking at you, Sessions), while others are strict but efficient (hello, JWTs). Let’s break it down.

### Session-Based Authentication

<b>How It Works</b>

- The server creates a session for each user and stores it (usually in memory or a database like Redis).
- The client gets a session ID (often in a cookie) and sends it with every WebSocket request.
- The server checks the session ID to see if the user is allowed in.

<b>Pros </b>
- <b>Easy to Implement:</b> If you’re already using sessions in your app, this is a no-brainer.
- Server-Side Control: You can easily invalidate sessions if something goes wrong (like a user logging out or getting hacked).

<b>Cons </b>
- <b>Not Scalable: </b> Storing sessions can become a bottleneck as your app grows. Imagine trying to manage a million clingy friends—it’s exhausting.
- <b>Stateful: </b> Sessions are tied to the server, which doesn’t play well with distributed systems.

Use sessions if you're building a small app, don’t need to scale horizontally, and want server-side control over authentication.

![Session Based Authentication](/images/posts/session_based_auth.png)


### JWT (JSON Web Tokens)

<b>How It Works</b>
- The server issues a signed token (JWT) to the client after authentication.
- The client sends this token with every WebSocket request.
- The server verifies the token’s signature and checks if the user is allowed in.

<b>Pros</b>
- <b>Stateless: </b> No need to store session data on the server. The token contains all the info the server needs.
- <b>Scalable: </b> Perfect for distributed systems. You can add more servers without worrying about session storage.
- <b>Flexible: </b> You can include custom claims in the token (like user roles or permissions).

<b>Cons</b>
- <b>Harder to Invalidate: </b> Once a JWT is issued, it’s valid until it expires. If you need to revoke it early, you’ll need to implement a blacklist or use short expiration times.
- <b>Larger Payload: </b> JWTs can be bigger than session IDs, which might increase bandwidth usage.

![JWT Based Authentication](/images/posts/jwt.png)

_JWTs are like a VIP pass. They’re self-contained, easy to carry around, and get you into the club—no questions asked._

Use JWTs if you're building a scalable, distributed system and want stateless authentication, even if it means dealing with a bit of extra complexity.


## WS vs WSS: The Difference Between a Postcard and a Locked Safe
When it comes to WebSocket connections, the difference between WS and WSS is like the difference between sending a postcard and locking your message in a safe. One is convenient but risky, and the other is secure but requires a bit more effort. Let’s break it down.

### WS (WebSocket): The Postcard

<b>What It Is</b>
- The unencrypted version of WebSocket.
- Data is sent in plain text over the wire.

<b>Pros </b>
- <b>Easy to Set Up:</b> No need for SSL/TLS certificates.
- <b>Great for Development:</b> Perfect for testing and local environments.

<b>Cons</b>
- <b>Insecure:</b> Data can be intercepted and read by anyone on the network.
- <b>Not for Production:</b> Using WS in production is like shouting your secrets in a crowded room—everyone can hear you.

If you’re using WS, attackers can easily eavesdrop on your WebSocket communication, steal sensitive data, or even inject malicious code. It’s a hacker’s dream and your worst nightmare.


### WSS (WebSocket Secure): The Locked Safe
<b>What It Is </b>

- The encrypted version of WebSocket, using SSL/TLS.
- Data is encrypted before being sent over the wire.

<b>Pros</b>

<b>Secure:</b> Even if someone intercepts the data, they can’t read it without the encryption key.
<b>Required for Production:</b> Most browsers and clients enforce WSS for secure connections.

<b>Cons</b>
- Slightly Harder to Set Up: You need an SSL/TLS certificate.
- Adds a Small Overhead: Encryption increases the size of the data and adds a bit of latency.

WSS ensures that your WebSocket communication is private and tamper-proof. It’s the only way to protect sensitive data and comply with security standards like GDPR.

_Pro Tip: If you’re deploying your app, get an SSL/TLS certificate (many providers like Let’s Encrypt offer them for free) and configure your server to use WSS._

## Additional Security Considerations
Securing WebSocket connections doesn’t stop at authentication and encryption. Here are some extra measures to lock things down:

- <b class="span_highlight">Rate Limiting:</b> Limit the number of connections or messages a client can send. Prevents abuse and DDoS attacks.
- <b class="span_highlight">Message Validation:</b> Validate and sanitize incoming messages to prevent injection attacks and ensure data integrity.
- <b class="span_highlight">CORS Policies:</b> Restrict which domains can connect to your WebSocket server to prevent unauthorized access.
- <b class="span_highlight">Heartbeat and Ping/Pong:</b> Detect and handle disconnected clients to prevent resource leaks and keep connections alive.
- <b class="span_highlight">Logging and Monitoring:</b> Track connections, disconnections, and errors to detect suspicious activity and debug issues.



## Conclusion

Securing WebSocket connections is like building a fortress—you need strong gates (authentication), a solid moat (encryption), and extra layers of defense (rate limiting, message validation, etc.). Whether you’re scaling with JWTs or keeping it simple with sessions, or locking down your data with WSS, the goal is the same: to build a secure, reliable, and performant application.

Don’t cut corners when it comes to security. After all, the only thing worse than a hacked app is explaining to your boss why it happened. So, go ahead, implement these measures, and make your WebSocket connections as secure as Fort Knox. Your users (and your sanity) will thank you.



