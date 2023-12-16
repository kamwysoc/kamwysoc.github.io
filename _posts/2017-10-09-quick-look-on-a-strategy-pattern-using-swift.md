---
layout: post
title: 'Design Patterns with Swift: Quick look at a Strategy Pattern'
date: 2017-10-09 17:40
category: blog
star: false
author: kwysocki
description: Quick look at Strategy Pattern using Swift
tag:
- swift
- iOS
- programming
- design patterns
---

![Image Title](/assets/posts/quick-look-on-a-strategy-pattern-using-swift/strategy.jpg)


Let's take a quick look at one of the design patterns that should help us to write a good Object-Oriented code.
The basic assumption of Strategy Pattern is that you can define many implementations that will conform to the protocol.

Take a look at a simple example that can be used on iOS applications.


Firstly, create a `protocol` which contains a method. In our case it will be:

## Define protocol

<script src="https://gist.github.com/kamwysoc/51c2bef4373a063aeafc9d9cb605d9e9.js"></script>

## Create strategies

Ok, most of the iOS apps use an `UIImage` to represent images in applications. The `UIImage` instance can be used to produce two different data representations of image `UIImagePNGRepresentation` and  `UIImageJPEGRepresentation`. Let's create classes that handle this stuff.

<script src="https://gist.github.com/kamwysoc/69fbffb20630cd273ed84a5ee2149f90.js"></script>

Now, as you can see - both classes conforms to the `ImageRepresentation` protocol but they differ in implementation. Each class represents a different **strategy**.

## Create client

The last thing - creating a client that uses one of the `ImageRepresentation` strategies.

<script src="https://gist.github.com/kamwysoc/5b44740021d9bba904cda4de47939e94.js"></script>

## Usage

<script src="https://gist.github.com/kamwysoc/48ee90472babb961ab789966d7e2ed7a.js"></script>

## Conclusions

The cool thing about Strategy Pattern is that we can change our strategy at runtime.
While using the Strategy Pattern we definitely conform to "Open-Close" SOLID principle. Our client is open for extensions by changing the strategy without changing the client implementation(close for modification). Also, the `ImageRepresenter` with Strategy Pattern included will be easiest to test.

Let's think how the above code could look like without Strategy Pattern:

### Using Switch

<script src="https://gist.github.com/kamwysoc/5be7d283e6e08052683af1c79405ce91.js"></script>

### or using multiple functions

<script src="https://gist.github.com/kamwysoc/8c6b66a014629604963b05799ab2a980.js"></script>

Both of these solutions definitely are not on the same line with CleanCode. Also, it might be hard to maintain that kind of code. The switch statement can grow with the next cases - what if we had to handle a 10, 20 or 100 strategies? The second one using multiple functions is also bad because we will continue duplicating the similar methods to handle each case. This few arguments should convince you to use Strategy Pattern. And last but not least, this two examples breakes the Open-Close principle.

This post was primarly posted on my company [blog](https://brightinventions.pl/blog/quick-look-on-a-strategy-pattern-using-swift/)
