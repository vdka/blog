---
title: State Restoration for Settings Changes
author: Ethan Jackwitz
date: '2018-10-27'
categories:
tags:
  - iOS
  - Code
  - Xcode
  - Technical
---

# State Restoration with Custom Transition Modals

When requesting permissions and the user has in the past declined your application, the best thing you can do is open to your apps settings page. However if the user changes any of the permissions your app is granted the system will kill your app. This is a smart way to help ensure permissions are more easily enforced.
The issue comes when a user would expect to be able to toggle the permission on and go straight back to where they were.
</br>
<video style="width:49%" controls>
  <source src="/vid/WithoutStateRestoration.MP4" type="video/mp4">
</video>
<video style="width:49%" controls>
  <source src="/vid/StateRestorationDesiredBehaviour.MP4" type="video/mp4">
</video>
</br>

You can see on the left that the app when reopened doesn't go straight back into the modal screen that was displayed ready to be used. Instead the user has to manually navigate back to it. The right is far better.

This is done using [State Restoration](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/PreservingandRestoringState.html).
State restoration isn't exactly the simplest API to implement. Following is the alterations required to make it work in this example.

```swift
  // AppDelegate.swift
  func applicationDidFinishLaunching(_ application: UIApplication) {
+     if window?.rootViewController == nil {
+         let root = ConsumerMainViewController()
+         window?.rootViewController = root
+     } else {
+         window?.rootViewController?.setNeedsStatusBarAppearanceUpdate()
+     }
  }

+ func application(_ application: UIApplication, willFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool {
+     window?.makeKeyAndVisible()
+     return true
+ }

+ func application(_ application: UIApplication, shouldSaveApplicationState coder: NSCoder) -> Bool {
+     coder.encode(Bundle.main.buildNumber, forKey: "App-Build-Number")
+     return true
+ }

+ func application(_ application: UIApplication, shouldRestoreApplicationState coder: NSCoder) -> Bool {
+     let version = coder.decodeFloat(forKey: "App-Build-Number")
+     return version == Bundle.main.buildNumber
+ }

+ func application(_ application: UIApplication, viewControllerWithRestorationIdentifierPath identifierComponents: [String], coder: NSCoder) -> UIViewController? {
+     if identifierComponents.last == "CheckInModalNavigationController" {
+         let nav = UINavigationController()
+         nav.restorationIdentifier = "CheckInModalNavigationController"
+         nav.setNavigationBarHidden(true, animated: false)
+         return nav
+     }
+     return nil
+ }
```

```swift
  // TabViewController.swift
  init() {
      super.init(nibName: nil, bundle: nil)
+     self.restorationIdentifier = type(of: self).description()
+     self.restorationClass = type(of: self)
  }

// ...
+ extension TabViewController: UIViewControllerRestoration {
+     static func viewController(withRestorationIdentifierPath identifierComponents: [String], coder: NSCoder) -> UIViewController? {
+         let vc = TabViewController()
+         // The app's root view controller must be set after it's restoration for any following presentations to work.
+         AppDelegate.shared.window?.rootViewController = vc
+         return vc
+     }
+ }
```

```swift
  // CheckInViewController.swift

  init() {
      super.init(nibName: nil, bundle: nil)
+     self.restorationIdentifier = type(of: self).description()
+     self.restorationClass = type(of: self)
  }

  func checkIn() {
      let modal = CheckInModalViewController()
      let nav = UINavigationController(rootViewController: modal)
+     nav.restorationIdentifier = "CheckInModalNavigationController"
+     nav.setNavigationBarHidden(true, animated: false)
      self.present(nav, animated: true)
  }


+ extension CheckInViewController: UIViewControllerRestoration {
+     static func viewController(withRestorationIdentifierPath identifierComponents: [String], coder: NSCoder) -> UIViewController? {
+         return CheckInViewController()
+     }
+ }
```

```swift
  // CheckInModalViewController.swift
  init() {
      super.init(nibName: nil, bundle: nil)
+     self.restorationIdentifier = type(of: self).description()
+     self.restorationClass = type(of: self)
      // ...

+ override func willMove(toParent parent: UIViewController?) {
+     super.willMove(toParent: parent)
+     if let parent = parent as? UINavigationController {
+         parent.transitioningDelegate = self
+         parent.modalPresentationStyle = .custom
+     }
+ }

+ extension CheckInModalViewController: UIViewControllerRestoration {
+     static func viewController(withRestorationIdentifierPath identifierComponents: [String], coder: NSCoder) -> UIViewController? {
+         let vc = CheckInModalViewController()
+         // This isn't restored automatically
+         vc.overlay.effect = vc.backgroundVisualEffect
+         return vc
+     }
+ }
```

### Some Resources

- [State restoration for modal view controllers](http://aplus.rs/2013/state-restoration-for-modal-view-controllers) - [aplus.rs](aplus.rs)
- [State Restoration Tutorial: Getting Started](https://www.raywenderlich.com/1395-state-restoration-tutorial-getting-started) - [raywenderlich.com](raywenderlich.com)

