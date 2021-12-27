---
title: "Creating a macOS Statusbar Application"
date: 2021-12-27T19:06:44Z
draft: false
tags: ["macos", "swift", "swift-ui"]
---
In this post we will create a macOS statusbar application.
In order to follow this tutorial you will need following items:

- A device that is capable of running macOS Big Sur
- Xcode (I am using Version 13.2.1 (13C100))

### Create Project

- Open Xcode and create new project.
- Select App under macOS platform
![statusbar_1](/img/statusbar_01.png)

- Enter your product name (Statusbar)
- Select Team if you have one or let Xcode generate one (My developer account)
- Enter an organization identifier (Reverse domain - dev.gokhun)
- Select SwiftUI as interface and Swift as language
- Uncheck core data and tests for now

![statusbar_1](/img/statusbar_02.png)

### Add Statusbar

In order to add statusbar and menu items we need to add `NSApplicationDelegate`
to our application.

Right click to `Statusbar` folder in project navigator and create a new swift
file:

```swift
// AppDelegate.swift
import AppKit

class AppDelegate: NSObject, NSApplicationDelegate {
    // Leave here empty for now
}
```

Open `StatusbarApp.swift` file and fill with followings:

```swift
//  StatusbarApp.swift
import SwiftUI

@main
struct StatusbarApp: App {

    @NSApplicationDelegateAdaptor(AppDelegate.self) var appDelegate

    var body: some Scene {
        Settings { // Dont forget to change this line to Settings
            ContentView()
        }
    }
}
```

`NSApplicationDelegateAdaptor`will let us use `AppKit` in our SwiftUI based application.
Since our application will be in statusbar we will not use `WindowGroup`. Instead we will use
`Settings` which will give us a preferences menu bar item.

Return to `AppDelegate.swift` file and override `applicationDidFinishLaunching` method to add statusbar item:

```swift
// AppDelegate.swift
import AppKit

class AppDelegate: NSObject, NSApplicationDelegate {

    // We need to declare NSStatusItem here, otherwise it gets destroyed after
    // applicationDidFinishLaunching is called
    var statusBarItem : NSStatusItem!
    var statusBarMenu : NSMenu!

    func applicationDidFinishLaunching(_ notification: Notification) {

        // Returns the system-wide status bar located in the menu bar.
        let statusBar = NSStatusBar.system

        // Returns a newly created status item that has been allotted a specified space within the status bar.
        self.statusBarItem = statusBar.statusItem(withLength: NSStatusItem.squareLength)
        self.statusBarItem.button?.image = NSImage(systemSymbolName: "star.fill", accessibilityDescription: "Status bar icon")

        // An object that manages an app’s menus.
        self.statusBarMenu = NSMenu()
        self.statusBarMenu.addItem(withTitle: "Hello", action: nil, keyEquivalent: "")

        // Add menu to statusbar
        self.statusBarItem.menu = self.statusBarMenu
    }
}
```

Finally we need to add a property to our target. Open Xcode and select the top item
in Project navigator. Then select `TARGETS` menu and navigate to `Info` tab.

![application_is_agent](/img/application_is_agent.png)

Add `Application is agent (UIElement) YES` as seen in screen shot above. This makes our application
run in background and do not populate the menu bar (File, edit, view ... items near apple logo).

At this stage we should have a star with a single menu item in our statusbar.

![star](/img/star.png)

### Open Windows

In order to make our statusbar application more useful let's open some windows from
statusbar items programmatically. We will create `NSWindow` instances by clicking
statusbar menu items. In these windows we will use SwiftUI elements.

First create a SwiftUI view with following content:

```swift
// HelloView
import SwiftUI

struct HelloView: View {
    var body: some View {
        Text("Hello from statusbar")
            .frame(width: 200, height: 100, alignment: .center)
    }
}
```

Add following function to `AppDelegate.swift` file:

```swift
@objc func sayHello() {
    // Get focus from other apps
    NSApplication.shared.activate(ignoringOtherApps: true)

    // Create the frame to draw window
    let hello = NSWindow(
        contentRect: NSRect(x: 0, y: 0, width: 640, height: 480),
        styleMask: [.titled, .closable, .fullSizeContentView],
        backing: .buffered,
        defer: false
    )
    // Add title
    hello.title = "Hello!"

    // Keeps window reference active, we need to use this when using NSHostingView
    hello.isReleasedWhenClosed = false

    // Lets us use SwiftUI viws with AppKit
    hello.contentView = NSHostingView(rootView: HelloView())

    // Center and bring forward
    hello.center()
    hello.makeKeyAndOrderFront(nil)
}
```

To call this function we will add a selector to our statusbar item:

```swift
// Find the line where you add Hello item
self.statusBarMenu.addItem(withTitle: "Hello", action: #selector(sayHello), keyEquivalent: "")
```

Finally our `AppDelegate.swift` file looks like this:

```swift
// AppDelegate.swift
import AppKit
import SwiftUI

class AppDelegate: NSObject, NSApplicationDelegate {

    var statusBarItem: NSStatusItem!
    var statusBarMenu: NSMenu!

    func applicationDidFinishLaunching(_ notification: Notification) {

        // Returns the system-wide status bar located in the menu bar.
        let statusBar = NSStatusBar.system

        // Returns a newly created status item that has been allotted a specified space within the status bar.
        self.statusBarItem = statusBar.statusItem(withLength: NSStatusItem.squareLength)

        // systemSymbolName is taken from SF Symbols app that is available for macOS
        // https://developer.apple.com/sf-symbols/
        self.statusBarItem.button?.image = NSImage(systemSymbolName: "star.fill", accessibilityDescription: "Status bar icon")

        // An object that manages an app’s menus.
        self.statusBarMenu = NSMenu()
        self.statusBarMenu.addItem(withTitle: "Hello", action: #selector(sayHello), keyEquivalent: "")

        // Add menu to statusbar
        self.statusBarItem.menu = self.statusBarMenu
    }

    @objc func sayHello() {
        // Get focus from other apps
        NSApplication.shared.activate(ignoringOtherApps: true)

        // Create the frame to draw window
        let hello = NSWindow(
            contentRect: NSRect(x: 0, y: 0, width: 640, height: 480),
            styleMask: [.titled, .closable, .fullSizeContentView],
            backing: .buffered,
            defer: false
        )
        // Add title
        hello.title = "Hello!"

        // Keeps window reference active, we need to use this when using NSHostingView
        hello.isReleasedWhenClosed = false

        // NSHostingView lets us use SwiftUI views with AppKit
        hello.contentView = NSHostingView(rootView: HelloView())

        // Center and bring forward
        hello.center()
        hello.makeKeyAndOrderFront(nil)
    }
}
```

Here is the final result:
![final](/img/final.png)

### Summary

To summarize:

- We have used an adapter to add AppKit to our SwiftUI application.
- We have used another adapter to call SwiftUI view from AppKit NSWindow.
- We have set application target property to run it in background

Source code for this application can be found in [GitHub](https://github.com/ghokun/macos-statusbar-template)
