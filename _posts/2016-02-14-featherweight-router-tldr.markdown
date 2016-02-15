---
layout: post
title:  "Featherweight Router tl;dr"
date:   2016-02-14 15:44:24 +1100
published: false
---
You don't want to read a full introduction, it's very conceptual and will follow later. And it's too long. This is the opinionated essentials only version you are looking for.

Introducing [Featherweight Router](https://github.com/featherweightlabs/FeatherweightRouter). It's like a washing machine that separates laundry automatically.<!--more-->

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

A Unidirectional State Flow Store. ReSwift etc. Referenced here primarily so we don't have to talk about State any further.


### View Controllers

Nothing router specific here. Just make sure they are testable and isolated. Pass requirements via protocols at or after instantiation time.

### View Presenters

{% highlight swift %}
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
{% endhighlight %}

Inject the Store and other properties when creating a presenter. The presenter returns a view controller when called.

## How? (_Routing specific_)

Routers match and set routes, then pass the matched presenters to RouterDelegates for display.

### RouterDelegate\<T\>

![](/assets/router-delegate.png)

Wraps a presenter and provides hooks for updating the display of the current route.

{% highlight swift %}
let delegate = RouterDelegate(presenter: somePresenter(store))
{% endhighlight %}

Once you go full router, you'll hardly every use a Presenter with wrapping it in a RouterDelegate, so just go and combine them all in your next refactor.

{% highlight swift %}
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
{% endhighlight %}


### Router\<T\>

Nailed the delegates? Well done. Now get routing!

![](/assets/router.png)

Oh. Yeah. Not so fast, the base router doesn't actually do anything other than hold a collection of routing functions. Each router type is an extension to the Router. They take a router, modify it's behaviour and return a new router.  Onward then.


### Junction Router =~ UITabBarController

![](/assets/junction-router.png)

![](/assets/junction.gif)

Router junctions match against child routes and passes the array of direct descendants to setChildren, and the matched route to setChild.

{% highlight swift %}
let tabBar = UITabBarController()

let tabBarPresenter = RouterDelegate(
    getPresenter: { tabBar },
    setChild: { tabBar.selectedViewController = $0 },
    setChildren: { tabBar.setViewControllers($0, animated: true) })

let someJunctionRouter = Router(tabBarPresenter).junction([
    someRoute,
    anotherRoute,
])
{% endhighlight %}


### Stack Router =~ UINavigationController

![](/assets/stack-router.png)

![](/assets/stack.gif)

Router stacks match against children in the same way as junctions, but pass a nested stack of presenters to setChildren instead.
 
{% highlight swift %}
let stack = UINavigationController()

let navigationPresenter = RouterDelegate(
    getPresenter: { stack },
    setChild: { _ in },
    setChildren: { stack.setViewControllers($0, animated: true) })

let someStackRouter = Router(navigationPresenter).stack([
    someRouteWithChildren,
    anotherRouteWithChildren,
])
{% endhighlight %}



### Named Route -> UIViewController

![](/assets/route.png)


And lastly, Named Routes, the endpoints of a matched route. Construct a route with a presenter, then assign it a name and optionally some children.

Name patterns are evaluated as regex matches and can be used for selecting id's or values from URL segments. The matched segment values are not passed to the delegate automatically, the ViewModels are expected to derive the values from state.

{% highlight swift %}

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

{% endhighlight %}

## Putting it all together

Last thing left is to scaffold out a shell for our app. The following code only covers the basic use case to illustrate where router creation happens and how it fits into the coordinator and initialisation. We will start from the AppDelegate and work our way down.

Lines marked with `external` are outside the scope of this intro, as opinions and framework choices for those components will vary per project. As are the `ViewControllers` and `ViewModels`.

{% highlight swift %}

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

{% endhighlight %}

Your move reader. Go flesh out those mockPresenters into something amazing.
