---
title: "Quick Start: Hot reloading in Swift"
layout: post
date: 2016-02-22 08:32:12 +1100
tags:
  - Beginner
  - Here be dragons
  - Dragons of AWESOME!
---

> Getting started with hot reloading inside a swift project.

Hot-reloading swift classes really is quite simple, thanks to [John Holdsworth](https://github.com/johnno1962)'s AWESOME [Injection Plugin for Xcode](https://github.com/johnno1962/injectionforxcode).

<video src='/assets/swift-hot-reload.mp4' preload='meta' controls></video>

## The TL;DR:

1. Clone the repo: [https://github.com/johnno1962/injectionforxcode](https://github.com/johnno1962/injectionforxcode)
2. Open in Xcode and run **Build**
3. Restart Xcode, and click 'Load Bundles'
4. Restart Xcode again
5. Open a project
6. Add a new Objective-C file called `main.m` (if one doesn't already exist)
7. Also, add the bridging header when prompted
8. From the menu select **Product > Injection Plugin > Patch Project for Injection**
9. From the menu: **Product > Injection Plugin > Tunable App Parameters** and enable **File Watcher** (and Inject Storyboards if you use them)
9. Recompile and run the project
10. Open a swift file, make changes and save
11. **TADA**. The file will be recompiled and injected into the running app

## Notes:

The injection only works for objects that can be swizzled. Which means only classes that inherit from `NSObject`. If you add or remove classes or files, the project will also need to be rebuilt.

If you have issues with file not found errors in the console when saving, check that Xcode is set to store derived data in the default location.

To perform actions on classes when they are reloaded, add an injected function to them:

```swift
class SomeClass: NSObject {
    func injected() {
        // I will run when my class is reloaded
    }
}
```

This will only be called on the class that is replaced. If you want to receive notifications whenever a class is changed you can use NSNotificationCenter:

```swift
class SomeClass: NSObject {
    init() {
        // ...
        NSNotificationCenter.defaultCenter().addObserver(self, selector: "interjected:", name: "INJECTION_BUNDLE_NOTIFICATION", object: nil)
    }

    func interjected(notification: NSNotification) {
        guard notification.name == "INJECTION_BUNDLE_NOTIFICATION" else { return }
        print("INTERJECTED")
    }
}
```

## Wrap up

You will definitely also want to check out [KZPlayground](https://github.com/krzysztofzablocki/KZPlayground) for using hot-reloading and adjustable parameters on the fly and [ReSwift](https://github.com/ReSwift/ReSwift) for replaying the event and calculating state after reloading.

If you make something interesting with it, let me know: [@karlbowden](https://twitter.com/karlbowden).

