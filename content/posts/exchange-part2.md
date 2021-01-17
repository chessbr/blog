+++
title = "Building an Exchange in Rust - Part 2"
categories = ["architecture"]
tags = ["architecture", "software engineering", "exchange", "trade system", "rust"]
summary = "Developing the Matching Engine."
date = "2021-01-18"
draft = true
+++

Hello and welcome once again. This is the part 2 of my experience of building an exchange using Rust.

Today I want to present some progress I've done on the project.

To start simple, my plan is to first build the core business of the project: the Matching Engine.

This service is responsible for receiving orders and match them against the order book. If an order is executed, we produce some output, otherwise the order goes to the book as well.

The first thing I had to do is to prepare the development environment. I created a workspace in Rust following [the documentation guidelines](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html).

I created a workspace and created a package named `matching-engine`. This package contains a lib and a binary. The lib will contain all the business logic and the binary will depend on the lib, this way, it would be easier to create automated tests over the lib functionalities.

The binary is super super simple, it's job is to receive an asset identifier through process arguments and then iteratively asking for order commands through the user input, like this:

```
> matching-engine APPL
Starting matching engine for APPL

Enter your order command in the format: BUY|SELL QTY PRICE
> BUY 10 100.50
BUY 10 at 100.50 added to the order book

Enter your order command in the format: BUY|SELL QTY PRICE
> SELL 15 110
Executed: 10 sold at 100.50
SELL 5 at 110.0 added to the order book
```

This is the expected output of the binary. Super simple. To develop the lib I will use the [TDD](https://en.wikipedia.org/wiki/Test-driven_development) technique.

You can find the source code of the project at [my GitHub](https://github.com/chessbr/rust-exchange).

The snippet for the binary CLI [is located here](https://github.com/chessbr/rust-exchange/blob/master/matching-engine/src/main.rs).

So far, so good, now let's start the hard work.
