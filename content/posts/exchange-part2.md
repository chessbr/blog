+++
title = "Building an Exchange in Rust - Part 2"
categories = ["architecture"]
tags = ["architecture", "software engineering", "exchange", "trade system", "rust"]
summary = "Developing the Matching Engine."
date = "2021-02-15"
draft = false
+++

Hello and welcome once again. This is the part 2 of my experience of building an exchange using Rust.

Today I want to present some progress I've done to the project.

To start simple, my plan is to first build the core business of the project: the Matching Engine.

This service is responsible for receiving orders and match them against the order book. If an order is executed, we produce some output, otherwise, the order goes to the book as well.

The first thing I had to do is to prepare the development environment. I created a workspace in Rust following [the documentation guidelines](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html).

I created a workspace and created a package named `matching-engine`. This package contains a lib and a binary. The lib will contain all the business logic and the binary will depend on the lib, this way, it would be easier to create automated tests over the lib functionalities.

The binary is super simple, it's job is to receive an asset identifier through process arguments and then iteratively asking for order commands through the user input, like this:

```
> matching-engine APPL
*** Starting matching engine for APPL ***

Enter your order command in the format: BUY|SELL QTY PRICE

BUY 10 100.50
Order QUEUED => 10 x 100.5

Enter your order command in the format: BUY|SELL QTY PRICE

SELL 15 110
Order QUEUED => 15 x 110.0

Enter your order command in the format: BUY|SELL QTY PRICE

SELL 10 99
Order EXECUTED => 10 x 100.5
```

This is the expected output of the binary. Super simple. To develop the lib I will use the [TDD](https://en.wikipedia.org/wiki/Test-driven_development) technique.

You can find the source code of the project on [my GitHub](https://github.com/chessbr/rust-exchange).

The snippet for the binary CLI [is located here](https://github.com/chessbr/rust-exchange/blob/v0.1.0.dev/matching-engine/src/cli.rs).

So far, so good, now let's start the hard work. First of all, I will design a basic library I want to use. Let's create a struct that will contain the matching engine state and some methods to add orders into the book.

Here is the initial version (for the sake of simplicity, I hid other methods, but the full code you can find [here](https://github.com/chessbr/rust-exchange/blob/v0.1.0.dev/matching-engine/src/engine/engine_impl.rs)):

```rs
pub struct MatchingEngine {
    asset: String,
    buy_order_book: Vec<Order>,
    sell_order_book: Vec<Order>,
}

impl MatchingEngine {
    pub fn new(asset: String) -> MatchingEngine {
        MatchingEngine {
            asset,
            buy_order_book: Vec::new(),
            sell_order_book: Vec::new(),
        }
    }
    pub fn add_order(&mut self, order_type: super::OrderType, quantity: u64, price: f32) -> Vec<OrderResult> {
        // supressed
    }
```

For now, the struct has an associated function to create a new matching engine instance (constructor) and a single method called `add_order` that will process a received order. It will return a `Vector` of results. The [OrderResult](https://github.com/chessbr/rust-exchange/blob/v0.1.0.dev/matching-engine/src/engine/order_result.rs) is basically a struct that describes what happened to the order inside the matching engine: either added it to the book or it was executed. If we have multiple outputs, then we have multiple results like partial executions (some amount was executed and other was queued, or even multiple executions with different prices).

Now that we have a basic API design we can start our TDD process. Let's start by creating a small test case to make sure the matching engine can fully execute an order (the received order can be fully executed at quantity and price):

A basic order testing from the [tests.rs](https://github.com/chessbr/rust-exchange/blob/v0.1.0.dev/matching-engine/src/engine/tests.rs) file.
```rs
#[test]
fn test_simple_order() {
    let mut engine = super::MatchingEngine::new(String::from("QWERTY"));

    let buy_order_results: Vec<super::OrderResult> = engine.add_order(super::OrderType::BUY,  100,  9.90);
    assert_eq!(buy_order_results.len(), 1);

    let buy_result: &super::OrderResult = &buy_order_results[0];
    assert_eq!(buy_result.result_type, super::OrderResultType::QUEUED);
    assert_eq!(buy_result.order_type, super::OrderType::BUY);
    assert_eq!(buy_result.quantity, 100);
    assert_eq!(buy_result.price, 9.90);

    let sell_order_results: Vec<super::OrderResult> = engine.add_order(super::OrderType::SELL, 100, 9.90);
    assert_eq!(sell_order_results.len(), 1);

    let sell_result: &super::OrderResult = &sell_order_results[0];
    assert_eq!(sell_result.result_type, super::OrderResultType::EXECUTED);
    assert_eq!(sell_result.order_type, super::OrderType::SELL);
    assert_eq!(sell_result.quantity, 100);
    assert_eq!(sell_result.price, 9.90);
}
```

This test shows us how the matching engine struct can be used to add orders and what output it produces.
We test whether a sell order will result in a single execution and the price will also match the queued buy order.

I added more tests and ran `cargo test`. It produced several errors but after the add test -> implement -> fix errors iteration I could reach a satisfying solution for the matching engine. It is working well and I am proud of it.

Learning Rust this way was really challenging, I spent a lot of time with simple language issues like wrong imports, packaging issues, etc, but nothing else. Once I could picture how Rust packages work I started to make real progress.

### Next steps

Now I want allow to sending orders to the Matching Engine orders through the network, so I can connect to a socket and dispatch the orders directly to it. I will probably use gRPC to communicate with my matching engine. Then I will write some Python/Node library to forward the orders as I want to create a small web app to send orders through a WebSocket.

I hope you like it!

See you, bye!
