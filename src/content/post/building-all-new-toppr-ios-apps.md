---
layout: ../../layouts/post.astro
title: Building the all-new Toppr iOS apps - Part 1
description: Building the all-new Toppr iOS apps - Part 1
dateFormatted: Aug 23, 2017
---

![Graphical rendering of the Toppr app running in Simulator](/assets/images/posts/toppr-app-running-in-sim.webp)

## Our Motivation to Start Over

Toppr began as a learning app which evolved into an adaptive platform where users can learn, practice, socialize and clear their doubts. Starting off as a prototype four years ago, Toppr soon became a popular mainstream product among students. We needed to reinvent our app to reflect the reality of 2017 and beyond.

## Our Approach

To harness the true possibilities of technology, we had to adopt a radical approach. That's why we decided to go back to the drawing board and completely redesign the Toppr app; with new features and improved technology.

Our old codebase was written in a different time, with a different mindset. Back then, we were a startup on the road to growth. Building a good, reliable product was the need of the hour.

Moving away from the old codebase would give us the freedom to make choices which we previously couldn't - choices that would have previously caused us to compromise on other aspects.

That's exactly what we did to create two sleek, beautiful and intuitive apps - [the Toppr app and the Doubts app](https://itunes.apple.com/us/developer/haygot-education-pvt-ltd/id958913153?mt=8) - for iOS. Creating these apps was a real fun journey.

Starting from scratch, we took our first steps by choosing…

## Swift

<p align="center" width="100%">
    <img src="/assets/images/posts/swift.webp"> 
</p>

When Apple announced an exclusive programming language for macOS and iOS devices, developers around the world were thrilled with the potential prospects it brought. With excitement though, also came many questions.

- How easy would it be to create apps with Swift?
- How would it perform in production?
- What would the interoperability between Objective-C and Swift be?
- What was the future of Swift?

We initially had doubts of our own. Over time though, Swift proved itself to be the language of the future. Once we were convinced, we knew what we needed to start coding.

---

![Morpheus from Matrix](/assets/images/posts/morpheus.webp)

> Any developer will agree when I say that writing error-free code is nigh impossible. I guess this is what makes us human! However, that's still no excuse.

We have to make sure that every build we push is well tested before release. That's where CI comes to the rescue.

## CI: Continuous Integration

It's always good to have a system that runs unit test cases automatically - every time a new build is pushed - and notifies developers if the build fails. That's where Continuous Integration (CI) will help us improve our app continuously.

---

![Continous Integration flow using BuddyBuild](/assets/images/posts/buddybuild.webp)

CI is a development practice that requires developers to integrate codes into a shared repository several times a day. Each check-in is then verified by an automated build, allowing teams to detect problems early.

### [BuddyBuild](https://buddybuild.com/)

> **Buddybuild ties together and automates building, deploying and gathering feedback for mobile apps.**
> If you're developing a mobile application, and are looking for a mobile focused continuous integration, continuous deployment and iterative feedback solution that takes minutes to setup and get running, buddybuild is the right solution for you.

_Buddybuild is the tool that I chose to bring Continuous Integration into both Toppr and Doubts apps. It was really easy for me to configure it and have it working with Xcode._

## Our Design Pattern

If you're new to iOS development, you're most likely to use MVC. It is after all, the default approach to developing apps for iOS. At Toppr, it made sense for us to chose the MVC pattern as it would allow new joinees to easily understand the codebase.

![model-view-controller](/assets/images/posts/mvc.webp)

### MVC: Model-View-Controller

- It is a high-level pattern that classifies objects based on the roles they play in an application
- It is a compound design pattern, comprising of several elemental design patterns
- The MVC pattern in Cocoa can be found in Cocoa Bindings, Cocoa Document Architecture etc.

Here's some more information at [Cocoa Core Competencies](https://developer.apple.com/library/content/documentation/General/Conceptual/DevPedia-CocoaCore/MVC.html).

## Network Layer

If your app interacts with a server, you need a Network Layer. For us, the seamless use of Toppr across platforms (web and mobile) is an essential aspect of our product. This also depends on Network Layer.

We wanted some flexibility so that we could add or remove endpoints easily. Typed parameters were needed to assure code safety, auto-completion and validation. Logging was necessary for easy debugging of our network requests.

And finally, our Network Layer needed to be testable and well documented in order to keep things maintainable.

Once we knew what we wanted to build, we found ourselves in the process of writing our own Network Layer. That's when we stumbled upon this amazing framework.

## Moya

![https://github.com/Moya/Moya](/assets/images/posts/moya.webp)

> So the basic idea of Moya is that we want some network abstraction layer that sufficiently encapsulates actually calling Alamofire directly. It should be simple enough that common things are easy, but comprehensive enough that complicated things are also easy.

Moya's features allow you to create custom plugins and providers. You can also subclass its classes to add features that suit your need.

_One of the coolest features of Moya for me, was RxSwift._

<p align="center" width="100%">
    <img src="/assets/images/posts/rx-swift.webp"> 
</p>

A Reactive Extension for Swift, RxSwift enhances the functional perspective (filter, map, reduce etc.) of Swift by adding reactive extensions; making it Functional Reactive Programing.

> In computing, reactive programming is an asynchronous programming paradigm concerned with data streams and the propagation of change.

Rather than statically assigning values to the variables, you now start observing things that might change in the future.

<p align="center" width="100%">
    <img src="https://media.giphy.com/media/ffJiLLtCk5Am4/giphy.gif"> 
</p>

'Why', you ask? Well, we tend to write a lot of code in order to handle external operations like 'managing Tap events through IBActions'. We subscribe to keyboard notifications in order to handle position changes. We also use closure to handle data returned from network requests.

All of this increases code complexity.

RxSwift resolves this complexity and helps us maintain consistency across our codebase. It allows us to use signals instead of notifications, which are hard to test. We can also use blocks to avoid multiple conditional specifiers and delegates that take up a lot of space in code.

Before we could embrace the power of RxSwift across the whole codebase though, we wanted to test the waters with our Networking. It turned out to be fantastic!

The last component that went into our networking stack was Sockets.

## Sockets

For our Doubts app, we wanted to implement one-to-one sessions where users could chat with tutors and send texts and images to convey their doubts.

We needed to create a channel where messages would seamlessly transit between students and tutors. All such channels had to be fast, smooth and efficient at handling chat sessions; especially in large numbers.

Using HTTP to perform this task would have caused a lot of traction. That's why we chose Sockets.

![https://github.com/Moya/Moya](/assets/images/posts/websockets.webp)

Sockets allows real time communication between the client and the server. By using a different protocol than HTTP, it allows for bidirectional data flow. In HTTP, you constantly need to ask for new messages in order to receive them. On the other hand, Sockets can listen to the server and receive messages when they are available.

For this project, we have used this awesome library on GitHub [SocketIO](https://github.com/socketio/socket.io-client-swift). Here are some of its key features:

- Supports socket.io 2.0+ (For socket.io 1.0 use v9.x)
- Supports binary
- Supports Polling and WebSockets
- Supports TLS/SSL
- Can be used from Objective-C

Sockets is powering next-gen web applications like:

- Real time apps
- Chat apps
- Internet of Things
- Online multiplayer games

Sockets is currently one of the next big things, having been adopted by developers across the world.

---

## A Recap

You now know why we recreated both iOS apps using Swift. We also told you about the architecture we chose and how Network Layer was a vital component to create the foundation for both apps.

However, there is still so much more that we have yet to tell you about the things that went into engineering both apps.

## In Part 2

There's bucket-loads of technical information coming your way in our [next blog post](/post/part-2-building-all-new-toppr-ios-apps).

In it, we will talk about the various frameworks and features of Swift that allowed us to create a robust and seamless experience that you see in the new iOS apps.

Don't forget to read it!

---

If you happen to find this article insightful, please share it for others to find.
