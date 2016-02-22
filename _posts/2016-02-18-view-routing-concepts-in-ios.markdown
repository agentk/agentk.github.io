---
layout: post
title:  "View Routing Concepts in Swift"
date:   2016-02-18 13:16:15 +1100
tags:
  - Entry Level
  - Conceptual
---

> Back story of view routing in Swift. All code is conceptual and intended to illustrate concepts only.

It's uncommon to see a backend web framework without a router. It makes sense. The web is built around URL's that naturally branch out to routes.

Since the rise of frontend frameworks, routers have become a natural fit there too. Plus there is no shortage of choice for routers.

The function of a UI router is fairly simple:

- Formalise the **separation** of views components
- Compose components into a **structure** of routes and matchers
- **Handle URL changes** by matching it to a path of routes and returning the routes to a coordinator

That really is all there is to it, and it's something we have been doing for a long time with [coordinator](http://khanlou.com/2015/01/the-coordinator/) and [presenter](http://merowing.info/2016/01/improve-your-ios-architecture-with-flowcontrollers/) patterns. The router is essentially a set of helper functions for declaring these patterns.

It is especially helpful when combined with state and side-effect separation. With very specific advantages:

- Because components are now **isolated**, their structure can easily be rearranged
- The **concerns** of view components are **simplified** to presentation of state, and generating actions from user input
- Services and APIs now only interact with the state store
- Each section of the application is easily **testable AND replaceable**
- Presentation behaviour can be changed based on device or screen size, or swapped out entirely

At this point you can probably guess the frontend web frameworks that implement these patterns (in part or whole): Elm, Cycle, Redux, React, Angular, Vue, etc. Familiarity with these frameworks is not necessary though, as we are concerned more with how to apply the concepts behind them in our swift applications.

## Introducing Featherweight Router

The main main downside to implementing routers in iOS has been the amount of boilerplate code it requires. I have repeated the pattern often enough that it warranted moving into a project of it's own. Hence I [introduce](https://karlbowden.com/featherweight-router-tldr/) to you the [Featherweight Router](https://github.com/featherweightlabs/FeatherweightRouter) project. It was designed to be:

- **Simple** - it doesn't need to be complicated to be useful and powerful.
- **Purposefully has no dependencies** - the routing layer should be lightweight and easily replaced
- **Not concerned with State** - Although it's recommended to use a Unidirectional State Flow implementation such as ReSwift, the State Store should remain separate, replaceable and testable.
- **Not tied to presentation** - The router does control presentation, but is not concerned with HOW it is presented. You are free to use UIKit, AppKit, TVKit, etc.
- **Extendable** - The base router type does not implement any route handling behaviour, it's all done through Router creators.


Featherweight Router doesn't make any decisions for you outside of declaring a route structure. You still need to choose a UI framework, how to handle route changes and where to store state.

## Wrapping up

Want to learn more? Check out:

- The [code heavy tl;dr introduction](https://karlbowden.com/featherweight-router-tldr/)
- The [Featherweight Router](https://github.com/featherweightlabs/FeatherweightRouter) project on GitHub

My sources of inspiration:

- [The Coordinator](http://khanlou.com/2015/01/the-coordinator/) by [Soroush Khanlou](http://www.twitter.com/khanlou)
- [Improve your iOS Architecture with FlowControllers](http://merowing.info/2016/01/improve-your-ios-architecture-with-flowcontrollers/) by [Krzysztof Zabłocki](https://twitter.com/merowing_)
- [URLNavigator](https://github.com/devxoul/URLNavigator) by [Jeon Suyeol](https://github.com/devxoul)
- [Functional View Controllers](http://chris.eidhof.nl/posts/functional-view-controllers.html) by [Chris Eidhof](http://www.twitter.com/chriseidhof/)

With a special thank you to every person promoting the use of solid functional patterns.

