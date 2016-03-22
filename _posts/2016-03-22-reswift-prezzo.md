---
layout: post
title: "Sydney Cocoaheads ReSwift Prezzo"
date: 2016-03-22 23:26:33 +1100
excerpt: "Sydney Cocoaheads, March 17th, ReSwift talk notes"
tags:
  - ReSwift
  - Swift
  - Cocoaheads
background: greys
---

> What follows are the notes from my talk at the Sydney Cocoaheads meetup, March 17th on **Unidirectional State Flow in Swift - programming without paradox**.

It was such an encouraging group to talk to. Interestingly, most people followed along, but I wasn't expecting such a 'penny-drop' moment with the demo. Maybe I should have started with the demo?

Slide content has not been transcribed, and some slides have been left out for the sake of brevity. Links are listed inline where possible.

<div class='feature-image'><img src="/assets/paradox/paradox-slide-1.png"></div>

## Unidirectional State Flow in Swift <br> - programming without paradox

What am I doing? Encouraging experimentation, with a *lightening* introduction, to the principles of Unidirectional State Flow, and how they apply to application design in Swift.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-3.png"></div>

## What is Unidirectional State Flow?

It is a set of principles for controlling state handing in applications.

[Variations](http://martinfowler.com/eaaDev/EventSourcing.html) have been around for a while. The one we are interested in is [Redux](http://redux.js.org), the JavaScript framework by [Dan Abramov](https://twitter.com/dan_abramov). Other variations can also bee seen frameworks like [Elm](http://elm-lang.org) and [Cycle](http://cycle.js.org).

At a high level, the [base principles](http://redux.js.org/docs/introduction/ThreePrinciples.html) we are concerned about are

- That state only travels in one direction
- That state is immutable
- And that state is only modified by pure functions

The last two might sound contradictory, so, before we jump into the application of these principals, it helps to understand what we mean by State.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-4.png"></div>

# What is State?

> **state**, _noun_ <br>
  the particular condition that someone or something is in **at a specific time**.

Notice those words? _At a specific time_. They are very important words that are often missed in the definition of State. Because:

> ### Although state changes over time, <br> the state at any particular point in time is fixed.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-5.png"></div>

Therefore, the whole idea of creating an application revolves around time streams.

An application is, at it's heart, a representation of a stream of states through time, that a user can interact with.

But we all write applications already. So what is the big deal then?

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-6.png"></div>

When learning application development, there is so much minutiae to deal with.
And as programmers, we are often stubborn, and impatient.

Because we have so many other things to remember, we want to get concepts up and running quickly. Which leads to either not knowing how to, or not getting around to refactoring for maintainability.

It leads to state in your View. But we know that in MVC we can solve this by refactoring. And now you have Massive View Controllers.

But we can refactor that too. And now there's state in all the things.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-7.png"></div>

## Hasn't this already been solved?

**Yes**, we have MVVM!

But MVVM is about the separation of concerns, between views and models - and that includes state, but it does not specify how to handle, and interact with, a stream of states over time.

By now you have probably guessed, this sounds a lot like an advertisement for Functional Reactive Programming.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-8.png"></div>

And that is because your right.

**FRP** is all about dealing with a **stream of values over time** in a **functional and declarative** way.

Remember our definition of an application? An application is a representation of **state over time**.

We see these abstractions a lot in other fields too, such as Physics.

Speed for example, represents a measurement of distance over time.
With time being constant and moving forward.

Think about the concepts and how they relate:

> ## Speed is distance over time <br> Like an Application is state over time

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-9.png"></div>

So..... If we already have reactive frameworks, then why do we need ReSwift?

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-10.png"></div>

As soon as you have multiple sources or streams of state, there is nothing stopping them from mutating, deviating and creating forks in the continuum.

Now you have a potential paradox.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-11.png"></div>

And by paradox, we mean **this**. The state stream has forked somewhere and now both these the components are doing what they are told, but displaying different state.

Sigh.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-12.png"></div>

If they are streams can't we just join them back up?

You could *if* you have **paradox driven tests**, but until then, I think we all know how dangerous it is to cross the streams.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-13.png"></div>

ReSwift is FRP for your *whole application state*, but simplified into Unidirectional State Flow principals.

In essence, it is the **theory of relativity in a framework**.

> And it is: **programming without paradox**

But just like time travel, it introduces an abundance of terms that can seem daunting to start with.

State, Store, Actions, Reducers, Dispatchers, Subscribers, Middleware, etc

So lets break them down and see how they all fit together.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-14.png"></div>

Luckily a picture speaks a thousand words.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-15.png"></div>

Most importantly, this is your **store**. It **controls the flow of state over time**. It *is* your **TARDIS**. It has:

- One entry point for actions
- One port, that freezes time, for reducing / producing a new state
- And one exit point for subscribing to state updates

Each component of an application, can only interact with dispatch, and subscribe.

But you don't want to tie all of your application components directly to the store. That would defeat the purpose.

So using protocols each component can subscribed to, just a subset of the state that it is concerned about.

> #### Store: controls the flow of state over time

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-16.png"></div>

With our theory kinda on-track and store outlined, we need to divert and have a quick look at restructuring application components. In a Todo app for example we might have a List View and an Item View.

Each of these application components get divided into:

- **Input and Output**, which are **State** and **Actions**
- **Side effects**, such as API calls, get **initiated by Action Creators** and **pushed to the edges** into **Middleware and Services**
- **State mutation** is separated into **Reducers**. You'll remember, they are the ones that **pause time while they calculate a new state**

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-17.png"></div>

How do we decide what goes into State, Reducers and Action Creators?

The **application state** needs to provide enough information -- at any point in time -- to satisfy all of the application components displayed. So we analyse each of our components. Ask them:

- What information do we need to provide to it?
- And what actions does it produce?

This normally separates nicely into Input and Output, dependency protocols. You will probably recognise that it looks a lot like a View Model. When our component is connected to a view model that conforms to these protocols, it keeps it nicely isolated from the store.

For a Todo List we would likely have a list of todo items to display, a set of filters, such as Available, Overdue and Completed, and the currently selected filter.

The List would also produce actions -- user initiated by touch events -- such as when a user changes the filter, or selects an item.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-18.png"></div>

Each of these sub-divisions, State, Actions and Reducers, are composable and nestable.

Each are simplified down to a single responsibility, then stacked like lego to create the store.

> This means each component and store item is **mockable**, **replacable**, and **testable in isolation**.

> Both outside of the store, AND outside of view controllers.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-20.png"></div>

So far, I have shown a very over simplified explanation.

So before we continue let me preface the rest with some warnings.

I am **NOT** saying every application should be structured in this way. I am simply advocating experimentation.

ReSwift might be **too complex** or even **too simple** for your application, and sometimes even removing ReSwift is the right decision.

Suppose you were to go DIY and wanted to create your own store, lets look at the minimum you would need.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-21.png"></div>


From our history lessons:

- We know that the **State** and **Action** are **value types** that get passed around
- The **Reducer** and **Subscriber** are **function types**
- And the **Store** is just a protocol tying them all together, to **control the flow of State**.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-22.png"></div>

Then we just need to scaffold out a store.

It needs to be initialised with a **Reducer** and **initial state**

And components need to be able to **subscribe to state updates**, and **dispatch actions**.

That is it for the base Store. I know I glossed over the base types, we will see those in more detail in a future demo.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-23.png"></div>

### Finally, when is ReSwift a good fit?

When you have multiple components presenting the same same state, or when you have multiple sources of actions that can effect the state.

And good for pushing side effects out to the edges of an application, such as dealing with APIs that you call, or that can push back in events.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-24.png"></div>

### What is ReSwift not a good fit for?

Simple applications that can already be reasoned about in your head. They don't need to be made unnecessarily complicated.

Applications with large state -- such as binary data -- that should not all be kept in memory at once.

Applications where you would have to deal with large numbers of unthrottled actions. Because it would require pausing time and running reducers every time an action is dispatched.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-25.png"></div>

### Common Pitfalls

As you might have guessed, **reducers need to be fast**. You should not sort an array inside a reducer every time an action is dispatched.

Reducers should not cause side effects. Remember, time is paused. There are specific rules and safe guards to stop you from trying to dispatch a new action from inside a reducer and cause a paradox.

You still need to watch out for globals. They cause coupling, unknown side effects and make testing unnecessarily hard. And can easily be avoided using coordinators or flow-controllers.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-26.png"></div>

### Now for what you all came to see: the tooling around ReSwift.

Using a coordinator and router, lets us keep an application de-coupled, and recreate and jump to any stack of views in the application.

Then using the ReSwift Recorder for time travel we can record and replay actions and move through the time stream of an application.

My personal favourite is code injection - when I haven't broken it. Although it's not a part of ReSwift, it makes for a great combination when you can hot reload classes from both Objective C and swift. With KZPlayground you can also isolate view components and treat them like playgrounds inside the simulator.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-27.png"></div>

Do we have any good examples? Have a look at [Charter](https://itunes.apple.com/us/app/charter-mailing-list-client/id1082212697?ls=1&mt=8), by [Matthew Palmer](https://twitter.com/_matthewpalmer):

- Swift mailing list client for iOS
- Available in the App Store
- Open source by default, h/t Artsy
- [github.com/matthewpalmer/Charter](https://github.com/matthewpalmer/Charter)

> Most interestingly, he recently made the decision to remove ReSwift to reduce the barrier to entry to new contributors.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-28.png"></div>

### [バイクフリマアプリ RIDE](https://itunes.apple.com/jp/app/id1088140538) by Fablic, Inc

> "Bike Flea Market App - RIDE"

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-29.png"></div>

Now I need to make shout out to all our contributors. Especially [Ben](https://twitter.com/benjaminencz) and [Aleskander](https://twitter.com/ARendtslev) for generously allowing me opportunities to contribute during the early stages.

You may have heard of ReduxKit and Swift-Flow. They are the original projects by Aleksander and Ben that we merged into ReSwift.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-30.png"></div>

We welcome new contributors, and the discussion on GitHub is especially help full. Tell us what worked for you and what didn't.

It also turns out ReSwift is big in Japan. I have no idea what they are saying about it, but I would help to translate documentation.

For similar projects, have a look at [Delta](https://github.com/thoughtbot/Delta) by [Jake Craige](https://twitter.com/jakecraige) at [thoughtbot](https://twitter.com/thoughtbot)
and [SwiftFlux](https://github.com/yonekawa/SwiftFlux) by [Kenichi Yonekawa](https://twitter.com/yonekawa).

SwiftFlux is based on the Flux framework that the inspiration for Redux was based out of.

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-31.png"></div>

Where to next?

- Project and documentation on GitHub:
	- [github.com/ReSwift/ReSwift](https://github.com/ReSwift/ReSwift)
	- [ReSwift documentation](http://reswift.github.io/ReSwift/)
- [Benjamin Encz: Unidirectional Data Flow in Swift / Realm presentation](https://realm.io/news/benji-encz-unidirectional-data-flow-swift/)
- Have a look at the [Redux](https://github.com/reactjs/redux) project. The code is readable and easy enough to port to swift. As well as [Dan Abramovs excellent video series on EggHead](https://egghead.io/series/getting-started-with-redux).

---

<div class='feature-image'><img src="/assets/paradox/paradox-slide-32.png"></div>
