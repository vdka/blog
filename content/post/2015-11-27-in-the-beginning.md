---
categories:
- Swift
date: 2015-11-27T00:00:00Z
published: true
tags:
- Swift
title: In The Beginning
url: /2015/11/27/in-the-beginning/
---

Initialization and configuration can be a tricky topic in Swift. It can be difficult to know where to put things. This leaves plenty of people with stray implicitly unwrapped optionals laying around. Thankfully there can be a couple of neat tricks to tidy up your initialization code.

Implicitly unwrapped optionals are going to feel most familiar to Obj-C developers. These act like pointers insofar as they allow you to declare a property without worrying about setting the property immediately. Implicitly unwrapping an optional is the developer _promising_ you that whenever you access the optional it will never be `nil`.

```swift
class Method1: Example1 {
	var request: NSURLRequest!

	func setup() { //eg. viewDidLoad
		let request = NSMutableURLRequest()
		request.URL = NSURL(string: "https://api.github.com/users/vdka")!
		request.cachePolicy = .ReturnCacheDataElseLoad
		request.HTTPMethod = "GET"
		self.request = request
	}
}
```

There are a couple of problems with this.

1. A developer might expect to be able to access the request from another view controller before initialization in order to do something smart like begin loading the resource during a `segue`. No problem right, declaring `var request: NSURLRequest!` you have _promised_ this value would always be set when accessed. Well, it isn't set until we call the `setup()` method. The typical _setup_ method we see in iOS development is `viewDidLoad()`. Accessing this request here will cause a crash.
2. The initialization of the request is completely separate to the declaration. This makes our code less readable.
3. The request must be declared as a `var` as you modify it after you initialize it in order to set it up for use.

Lets look at a way we can address both of these issues.

```swift
class Method2: Example2 {
	var request: NSURLRequest {
		let request = NSMutableURLRequest()
		request.URL = NSURL(string: "https://api.github.com/users/vdka")!
		request.cachePolicy = .ReturnCacheDataElseLoad
		request.HTTPMethod = "GET"
		return request
	}
}
```

Here what we are doing is taking advantage of Swift's _computed properties_ feature. Essentially this is short hand for declaring a property with a getter.

In fact this can be expanded to be of the form:

```swift
var request: NSURLRequest {
	get {
		...
	}
}
```

What we have perfectly addresses the issues we had with the previous `Method1`. The request value is now _computed_ upon request and the initialization code is now a _part_ of the properties declaration.

This is perfect right? Well, not exactly.

So what is wrong with it then?

It is now a computed property. Meaning every time you access the property it is re-computed.

Want to access this request 400 times in a couple seconds? That will really hurt.

This can be seen by adding a count to each bit of configuration code and accessing the properties multiple times.

The real kick is when you realize that accessing the property on the class `Method1` will only ever call the configuration code once, when you call `setup()`. So how can we solve this?

Well there is an even swifter way to do this.

```swift
class Method3: Example2 {
	let request: NSURLRequest = {
		let request = NSMutableURLRequest()
		request.URL = NSURL(string: "https://api.github.com/users/vdka")!
		request.cachePolicy = .ReturnCacheDataElseLoad
		request.HTTPMethod = "GET"
		return request
	}()
}
```

This approach takes advantage of Swift's typing system and closures. The declaration of the request on this class may look like it is a closure type and not a NSURLRequest type (`(_) -> NSURLRequest`) and well it does. However, because of the closure calling itself after declaration it is actually _just_ the return type.

Thanks to this no longer being a computed property the configuration code will no longer be run upon every call.

Why stop there? To make things even better why not just run the configuration code when it is needed. This can be done as follows:

```swift
class Method4: Example2 {
	private(set) lazy var request: NSURLRequest = {
		let request = NSMutableURLRequest()
		request.URL = NSURL(string: "https://api.github.com/users/vdka")!
		request.cachePolicy = .ReturnCacheDataElseLoad
		request.HTTPMethod = "GET"
		return request
	}()
}
```

Note the need to add `private(set)` in order to maintain immutability to outside classes. I would love to see `lazy let` be a future feature.
