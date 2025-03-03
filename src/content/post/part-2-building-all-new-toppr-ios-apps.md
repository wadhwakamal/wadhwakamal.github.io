---
layout: ../../layouts/post.astro
title: Building the all-new Toppr iOS apps - Part 2
description: Building the all-new Toppr iOS apps - Part 2
dateFormatted: Oct 06, 2017
---

![Graphical rendering of the Toppr app running in Simulator](/assets/images/posts/toppr-dev-overview.webp)

As promised, [I am back with the sequel post to my last blog.](/post/building-all-new-toppr-ios-apps) If you haven’t read it, make sure that you do since I have discussed the architecture, network layer and the build process leading up to our foundation stack.

## When in Doubt, Follow the Protocol

While building the apps, we tried to effectively channelize ways in which students learn, practice and assess themselves through Goals, Question Banks, Tests (including Previous Years’ Papers) and more.

While the **Bookmark** and **Ask Doubts** features are shared in all of them, every other section has its own set of features which have been offered as required.

To achieve this, we can duplicate the code in order to avail the functionalities. This, however will result in a code duplication and loose code maintainability.

![Graphical rendering of the Toppr app running in Simulator](/assets/images/posts/lotr.webp)

Integrating this using one class and inheriting it in a subclass might work fine with the current implementation, but it could end up losing its flexibility and become difficult - if not impossible - to make changes to the application architecture in the future.

> The problem with inheritance is that it is made with the hierarchy of tightly coupled classes, and any change in the super-class will end up having a ripple effect on its subclass.

Object-oriented programming has been used for decades to solve computer problems by modelling them in classes. With Swift 2.0, Apple improved the functionality of Protocols. With it, came a new programming pattern — **Protocol Oriented Programming**.

Here are some really impressive things you can do with Protocols:

- You can extend the protocol to provide a default implementation
- You can inherit a protocol from other protocols and then add extra functionalities over the inherited functions
- You cannot inherit multiple classes, but can adopt multiple protocols

That’s why, instead of using inheritance, we created protocols for each feature with default implementation and implemented the in our classes, wherever required.

## The Curious Case of avoiding hard-coding

Hard-coding is generally considered an anti-pattern. People generally get furious when they see hard-coded values in code reviews.

![](/assets/images/posts/sparta.webp)

[As seen on Wikipedia](https://en.wikipedia.org/wiki/Hard_coding)

> While hard-coding is often required, it can also be considered an anti-pattern. Programmers may not have worked out a dynamic user interface solution for the end user but must still deliver the feature or release the program. This is usually temporary but does resolve the pressure to deliver the code in a short term sense. Soft-coding is later done to allow a user to pass on parameters that give the end user a way to modify the results or outcome.

It is generally not considered a good programming practice and it should be avoided, but every rule has its own exceptions.

Here are a few instances where hard-coding would be preferred:

- When the input value is going to remain constant (eg: PI)
- For memory allocation constraints (eg: Embedded Softwares)
- Cryptographic tokens
- Pre-processor constants etc.

Outside the above instances, we too avoided hard-coding using these amazing frameworks - [Swiftgen](https://github.com/SwiftGen/SwiftGen), which auto-generates assets, storyboards, Localizable strings etc. to get rid of string-based APIs and [Reusable](https://github.com/AliSoftware/Reusable), a Swift mixin for reusing views easily and in a type-safe way.

## [Swiftgen](https://github.com/SwiftGen/SwiftGen)

If you would like to programmatically initiate and present a view controller, you are dependent on string-based APIs, which takes a string, and return the respective view controller.

```
let storyboard = UIStoryboard(name: "MyStoryboardName", bundle: nil)
let controller = storyboard.instantiateViewController(withIdentifier: "someViewController")
self.present(controller, animated: true, completion: nil)
```

There are a few problems with this approach. You need to write the exact name of the storyboard and view controller you want to access. There is also the possibility of a typo.

In case someone changes the name of the storyboard or view controller, your code will still compile but might cause a crash at runtime.

To avoid these hassles, Swiftgen provides automatically generated enums which makes it easy to access the required view controller:

```
let controller = StoryboardScene.MyStoryboardName.someViewController.instantiate()
self.present(controller, animated: true, completion: nil)
```

Similarly, you can also avoid hard-coding for:

- Colors
- Fonts
- Strings and xcassets

## [Reusable](https://github.com/AliSoftware/Reusable)

<p align="center" width="100%">
    <img src="/assets/images/posts/reusable.webp">
</p>

While using enum constants is way better than using string literals to identify things uniquely but when it comes to identifying TableView or CollectionView Cell with reusable identifier, a better solution is Swift’s generics + Protocols, which is exactly what Reusable does.

> This library aims to make it super-easy to create, dequeue and instantiate reusable views anywhere this pattern is used - from the obvious UITableViewCell and UICollectionViewCell to custom UIViews; even supporting UIViewControllers from Storyboards.
> All of that, simply by marking your classes as conforming to a protocol without having to add any code, and creating a type-safe API with no more String-based API.

What it actually does is declaring reuseIdentifier as a static variable on UITableViewCell subclass, then using it to initialise cells of that class and using generics they implement the `dequeueReusableCell` method.

With Swift’s type inference, we can use the following code to instantiate our cell:

```
let cell: CustomCell = tableView.dequeueReusableCell(indexPath: indexPath)
```

---

## Layer of Persistence

When we started working on the Doubts app we wanted a robust, fast and simple persistent layer where we could store chats on demand and give the user a smooth scrolling experience.

We had SQL, Core Data and Realm as the options to work with. We realized that SQL was too complex for such a small task, while Core Data needs a huge boiler plate code.

_Realm has been around for sometime and has shown promising results. That’s why we considered this the perfect opportunity to integrate Realm in our persistence layer._

### Realm

![model-view-controller](/assets/images/posts/realm.webp)

Let’s start with an introduction first.

Realm Mobile Database is a cross-platform mobile database solution for iOS and Android. It is built to be better and faster than SQLite and Core Data. It’s faster, better, easier to use and will let you do many things with just a few lines of code.

[According to Realm](https://realm.io/products/realm-mobile-database/)

> Realm Mobile Database is an alternative to SQLite and Core Data. Thanks to its zero-copy design, Realm Mobile Database is much faster than an ORM, and often faster than raw SQLite.

Ever since its inception, developers have embraced this amazing framework due to the below reasons:

- Speed
- Simplicity
- No SQL

We used realm in our Doubts app to super-charge our chat interface and load a massive amount of messages. It was easy for us to filter, sort and map the live objects to get desired results without compromising on speed.

_We mentioned these important libraries to show our love for them, as they helped us achieve what we wanted in lesser time._

With this, I come to the end of this two-part discussion. I’ve given you some insights into the inner workings of the Toppr iOS journey.

Here’s hoping these insights help you uncover and embrace the innovative possibilities that lie ahead, just like we did.

---

If you happen to find this article insightful, please share it for others to find.
