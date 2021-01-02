+++
title = "Building an Exchange in Rust - Part 1"
categories = ["architecture"]
tags = ["architecture", "software engineering", "exchange", "trade system", "rust"]
summary = "How I designed and architected my exchange software."
date = "2021-01-01"
draft = true
+++

Welcome to my first post about designing and architecting software for an exchange. Today I will describe how I planned my solution for a trading system that can be used in exchanges with any kind of asset.

Before I dig down into the technical details of the solution I propose, I will first write down some features and requirements that my trading system should have and how it should work.

> **Addendum**: I have no experience with Rust, cryptocurrencies, or exchanges, this is a study case for learning purposes. If I am doing something very wrong here, please let me know, I would love to learn from you 😊.

### The specification

The simple definition of an Exchange is to connect people who want to buy something, to people who want to sell that same thing.

The software should receive orders to **buy** or **sell** a **quantity** of **something** for a **price**, or **according to some pricing rules**.

At this part, I am not interested in how the money will flow neither if the user has the proper money to buy the asset. The settlement process will be discussed later. For now, I just want to enable users to send their orders to the trading system and the orders can be executed when the system finds a match.

The software will be incrementally improved until we reach a point that we have nearly all the features that a real exchange should have: settlement, risk management, user authentication, etc.

You can argue that the correct way of doing this is architecting the entire system at once and then develop it incrementally. Well, as this project is for learning purposes, this is the best way I can think of to learn what is required and also to learn from the errors that I will probably do. It is a prototype that I will use to understand what is good and what it's not, then I can start over, doing things right (at least better than the previous version).

From a technical perspective, the trading system I want to build should:

- use microservices: every computing node should be responsible for a single task
- be scalable: maybe the harder part of the infrastructure, I have some ideas but we'll explore this later
- support at least 100 requests/sec with a response time ~10ms, it means a hundred users sending their orders, per second - does this sound crazy?
- be asset type agnostic: support stocks, cryptocurrencies, derivatives, etc.
- be accessible through TCP sockets: sending orders using a very simple protocol
- fairly notify all users about the orders executed (trades): all trades ticks should be sent to all interested parties at the "same" time
- store logs of all orders received and also all orders executed
- be developed using Rust: I just want to learn this language, that's it

These are all the requirements I have in my mind for the first version of the software, and it is already quite a lot!

### The data flow

In this section, I will describe how the information is going to flow throughout the system.

{{< figure src="/img/exchange-part1/data-flow.png" alt="Data flow" >}}

Now let us explore the services we need and how they deal with the information they receive.

##### Order Handler

This is the service that will communicate with the users directly through a TCP connection using a simple protocol. The user connects to the service host and submits a payload through the socket. Once the order is successfully received (and logged), the user received a response. This communication should be fast (~10ms) as all the hard work (storing into the database and matching orders) will be done asynchronously.

The order handler will read the asset from the payload and it will enqueue the order to the correct queue - every asset should have its own queue.

The service won't return whether the order was executed or not, it is not its responsibility. The user must register to the Market Data Notifier service to receive notifications about his order.

##### Order Processing Queue

The order receiver will log the request to the database and notify the correct Matching Engine that a new order message was received.

##### Matching Engine

The Matching Engine will consume the order messages and processes them. When a buy order matches one or more sell orders (or vice-versa), the Matching Engine sends the resulting trades to the Trade Processing Queue and updates the order book.

The order book is stored in memory to prevent I/O overheads like writing to the disk or waiting for a network request.

##### Trade Processing Queue

This service will receive messages from the Matching Engine about orders executed, canceled, or changed. The service will then log the message to the database and will dispatch a message to Market Data Notifier.

##### Market Data Notifier

This service will have thousands of users connected waiting for order notifications. The service will broadcast the trades for the users. Also, users can register to receive notifications about their own orders.

---

This is the first version of the project's architecture that I am going to use for the next steps. It can be all wrong, but I will discover later why.

I hope you liked it. Let me know your thoughts about this initial architecture. Is it too simple? Too complex? Won't work? :smiley:

#### References

Here are some nice videos I watched to get some context about how to build everything.

- https://www.youtube.com/watch?v=b1e4t2k2KJY
- https://www.youtube.com/watch?v=dUMWMZmMsVE
- https://www.youtube.com/watch?v=yBNpSqOOoRk

Thank you! See you. :rocket:
