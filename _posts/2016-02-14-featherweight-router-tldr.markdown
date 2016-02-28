---
layout: post
title:  "Featherweight Router tl;dr"
date:   2016-02-14 15:44:24 +1100
excerpt: "Overview of core classes in Featherweight Router."
tags:
  - Terse
  - Code heavy
background: greys
---

You don't want to read [background story](https://karlbowden.com/view-routing-concepts-in-ios/), it's very conceptual, and it's too long. This is the opinionated, essentials only version you are looking for.

Introducing [Featherweight Router](https://github.com/featherweightlabs/FeatherweightRouter). It's like a washing machine that separates laundry automatically.

## What?

A collection of functions and patterns (coordinators and presenters) for:

- Separating view controllers
- Constructing routers
- Matching a route to a path of view controllers
- Updating the presentation of view controllers

## Why?

To maintain a separation of concerns.

## How? (_Items not specific to routing_)

### Store

> Store: dispatcher of actions and returner of state

A [Unidirectional State Flow](https://realm.io/news/benji-encz-unidirectional-data-flow-swift/) Store. [ReSwift](http://reswift.github.io/ReSwift/) etc. Referenced here primarily so we don't have to talk about `State` any further.


### View Controllers

Nothing router specific here. Just make sure they are testable and isolated. Pass requirements via protocols at or after instantiation time.

### View Presenters

```swift
typealias Presenter = () -> ViewController
typealias PresenterCreator = T -> Presenter

// ie
func somePresenter(store: Store) -> Presenter {

    var viewController: ViewController!

    return {
        if viewController == nil {
            viewController = SomeViewController(
                viewModel: SomeViewModel(store: store))
        }
        return viewController
    }
}
```

Inject the `Store` and other properties when creating a presenter. The presenter returns a view controller when called.

## How? (_Routing specific_)

Routers match and set routes, then pass the matched presenters to `RouterDelegate`s for display.

### RouterDelegate: `RouterDelegate<T>`

```swift
public struct RouterDelegate<T> { // Cliffs notes edition
    public var presenter: T
    public func set(child: T)
    public func set(children: [T])
}
```

Wraps a presenter and provides hooks for updating the display of the current route.

```swift
let delegate = RouterDelegate(presenter: somePresenter(store))
```

Once you _go full router_, you'll hardly every use a Presenter without wrapping it in a `RouterDelegate`. So just go and combine them all in your next refactor.

```swift
func someLazyPresenter(store: Store) -> RouterDelegate<ViewController> {

    var viewController: ViewController!

    return RouterDelegate() {
        if viewController == nil {
            viewController = SomeViewController(
                viewModel: SomeViewModel(store: store))
        }
        return viewController
    }
}
```


### Router: `Router<T>`

Nailed the delegates? Well done. Now get routing!

```swift
public struct Router<T> { // Cliffs notes edition
    public var delegate: RouterDelegate<T>
    public var presenter: T
    public var handlesRoute: String -> Bool
    public var getStack: String -> [T]?
    public var setRoute: String -> Void
    public init(_ delegate: RouterDelegate<T>,
        handlesRoute: (String -> Bool)? = nil,
        setRoute: (String -> Void)? = nil,
        getStack: (String -> [T]?)? = nil)
}
```



Oh. Yeah. Not so fast, the base router doesn't actually do anything other than hold a collection of routing functions. Each router type is an extension to the `Router`. They take a router, modify it's behaviour and return a new router. Onward then.


### Junction Router `=~ UITabBarController`

![](/assets/junction.gif)

```swift
extension Router { // Cliffs notes
    public func junction(junctions: [Router<T>]) -> Router<T>
}
```

Router junctions match against child routes and passes the array of direct descendants to setChildren, and the matched route to setChild.

```swift
let tabBar = UITabBarController()

let tabBarPresenter = RouterDelegate(
    getPresenter: { tabBar },
    setChild: { tabBar.selectedViewController = $0 },
    setChildren: { tabBar.setViewControllers($0, animated: true) })

let someJunctionRouter = Router(tabBarPresenter).junction([
    someRoute,
    anotherRoute,
])
```


### Stack Router `=~ UINavigationController`

![](/assets/stack.gif)

```swift
extension Router { // Cliffs notes
    public func stack(stack: [Router<T>]) -> Router<T>
}
```

Router stacks match against children in the same way as junctions, but pass a nested stack of presenters to setChildren instead.
 
```swift
let stack = UINavigationController()

let navigationPresenter = RouterDelegate(
    getPresenter: { stack },
    setChild: { _ in },
    setChildren: { stack.setViewControllers($0, animated: true) })

let someStackRouter = Router(navigationPresenter).stack([
    someRouteWithChildren,
    anotherRouteWithChildren,
])
```


### Named Route `=~ UIViewController`

And lastly, Named Routes, the endpoints of a matched route. Construct a route with a presenter, then assign it a name and optionally some children.

```swift
extension Router { // Cliffs notes
    public func route(pattern: String) -> Router<T>
    public func route(pattern: String, children: [Router<T>]) -> Router<T>
}
```

Name patterns are evaluated as regex matches and can be used for selecting id's or values from URL segments. The matched segment values are not passed to the delegate automatically, the `ViewModel`s are expected to derive the URL segment values from state.

```swift

let someRouter = Router(aDelegate).route("animal", children: [
    Router(aDelegate).route("animal/mammal", children: [
        Router(aDelegate).route("animal/mammal/cow"),
        Router(aDelegate).route("animal/mammal/pig"),
    ]),
    Router(aDelegate).route("animal/fish", children: [
        Router(aDelegate).route("animal/fish/salmon"),
        Router(aDelegate).route("animal/fish/trout"),
    ]),
])

// Using regex's for matching
let someRouter = Router(aDelegate).route("animal", children: [
    Router(aDelegate).route("animal/\\w+", children: [
        Router(aDelegate).route("animal/\\w+/\\w+")
    ])
])

```

## Putting it all together

Last thing left is to scaffold out a shell for our app. The following code only covers the basic use case to illustrate where router creation happens and how it fits into the coordinator and initialisation. We will start from the `AppDelegate` and work our way down.

Lines marked with `external` are outside the scope of this intro, as opinions and framework choices for those components will vary per project. As are the `ViewController`s and `ViewModel`s.

```swift

import UIKit
import FeatherweightRouter

typealias Store = ExternalStore

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?

    func application(application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {

        window = UIWindow(frame: UIScreen.mainScreen().bounds)
        window?.rootViewController = appCoordinator()
        window?.makeKeyAndVisible()
        return true
    }

}

func appCoordinator() -> UIViewController {

    let store = appStore() // <- EXTERNAL
    let router = appRouter(store)
    store.subscribe() { state in // <- EXTERNAL
        router.setRoute(state.route)
    }

    store.setPath("some-initial-path")

    return router.presenter
}

func appRouter(store: Store) -> Router<UIViewController> {

    // UITabBarController
    return Router(tabBarPresenter()).junction([

        // Welcome tab: UINavigationController
        Router(navigationPresenter("Welcome")).stack([
            Router(mockPresenter(store, title: "Welcome")).route("welcome", children: [
                Router(mockPresenter(store, title: "Register")).route("welcome/register"),
                Router(mockPresenter(store, title: "Login")).route("welcome/login"),
            ]),
        ]),

        // About tab
        Router(mockPresenter(store, title: "About")).route("about"),
    ])
}

func tabBarPresenter() -> RouterDelegate<UIViewController> {

    let tabBarController = UITabBarController()

    return RouterDelegate(
        getPresenter: { tabBarController },
        setChild: { tabBarController.selectedViewController = $0 },
        setChildren: { tabBarController.setViewControllers($0, animated: true) })
}

func navigationPresenter(title: String) -> RouterDelegate<UIViewController> {

    let navigationController = UINavigationController()
    navigationController.tabBarItem.title = title

    return RouterDelegate(
        getPresenter: { navigationController },
        setChild: { _ in },
        setChildren: { navigationController.setViewControllers($0, animated: true) })
}

func mockPresenter(store: Store, title: String) -> RouterDelegate<UIViewController> {

    let viewController = MockViewController(MockViewModel(store: store, title: title))

    return RouterDelegate() { viewController }
}

```

Your move. These `mockPresenters` aren't going to turn into something amazing on their own.
