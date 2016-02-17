#ReactiveSecrets

I really like `ReactiveCocoa` and started using this framework in nearly all of my projects, however sometimes I had a hard time to find a reactive solution without help for easy looking tasks. Therefore I started this document to collect some use cases I stumbled across during development. This document should serves as a kind of cheat-cheet or as a best practice example collection.

Please note that this is not a pure beginners guide for `ReactiveCocoa` and requires that you have already read at least [Framework Overview](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/Documentation/FrameworkOverview.md), [Basic Operators](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/Documentation/BasicOperators.md) and
[Design Guidelines](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/Documentation/DesignGuidelines.md)

![Swift 2](https://img.shields.io/badge/Swift-2.1-orange.svg) ![ReactiveCocoa 4](https://img.shields.io/badge/ReactiveCocoa-4.0-green.svg)

# Table of Contents
1. [NSNotificationCenter](#NSNotificationCenter)
2. [Contributing](#Contributing)

## NSNotificationCenter

Adjust scrollview when keyboard appears / disappears the reactive way.


```
NSNotificationCenter.defaultCenter()
	.rac_notifications(UIKeyboardWillShowNotification, object: nil)
	.startWithNext { [weak self] notification in 
		if let keyboardSize = notification.userInfo?[UIKeyboardFrameEndUserInfoKey]?.CGRectValue.size, let weakSelf = self {
			var currentInset = weakSelf.tableView.contentInset
			currentInset.bottom = keyboardSize.height
	   	 	weakSelf.tableView.contentInset = currentInset
	   		weakSelf.tableView.scrollIndicatorInsets = currentInset
	 	}
	}

```

Be aware, that **you have to cancel the subscription when it´s not longer needed.** `[weak self]` or `[unowned self]` within the closure does not cancel the subscription when e.g. the viewcontroller gets popped or dismissed. 

Two possible solutions, as always there are more ways to do that..

**1. take care of your disposable**
	
```
let disposable = NSNotificationCenter.defaultCenter()
	.rac_notifications(UIKey..........
```
>
hold a reference to the returned disposable and use `disposable.dispose()` to cancel the subscription

+ (+) very transparent
+ (+) manual control of subscription
+ (-) requires some boilerplate (e.g. holding reference of disposable)

**2. use `takeUntil` operator in combination with `rac_willDeallocSignal`**

>
`rac_willDeallocSignal` works only on `NSObject`. As another constraint, this operator returns an oldschool `RACSignal` which must be mapped to an `SignalProducer`. Time for an extension:

```
extension NSObject {
	func rac_willDeallocSignalProducer() -> SignalProducer<(), NoError> {
    	return rac_willDeallocSignal()
    		.toSignalProducer()
    		.flatMapError { _ in .empty }
    		.map { _ in () }
	}
}
```

> Then you can do

```
NSNotificationCenter.defaultCenter()
	.rac_notifications(UIKeyboardWillShowNotification, object: nil)
	.takeUntil(self.rac_willDeallocSignalProducer())
```
+ (+) automatically cancels the subscription when current object deallocs
+ (+) no boilerplate (except of optional `NSObject` extension)
+ (-/+) doesn´t work if you have accidently a strong reference to that object (e.g. caused by `self` caputred within a closure)
+ (-) works only with `NSObject`


**Complete solution with ``takeUntil``**

```
NSNotificationCenter.defaultCenter()
	.rac_notifications(UIKeyboardWillShowNotification, object: nil)
	.takeUntil(self.rac_willDeallocSignalProducer())
	.startWithNext { [weak self] notification in 
		if let keyboardSize = notification.userInfo?[UIKeyboardFrameEndUserInfoKey]?.CGRectValue.size, let weakSelf = self {
			var currentInset = weakSelf.tableView.contentInset
			currentInset.bottom = keyboardSize.height
	   	 	weakSelf.tableView.contentInset = currentInset
	   		weakSelf.tableView.scrollIndicatorInsets = currentInset
	 	}
	}

```

# Contributing

If you want to help making this document a useful collection of `ReactiveCocoa` use cases, add new entries of your daily life reactive development or enhance existing examples 👍🏻
 

* [Fork it](http://help.github.com/forking/)
* Create new branch to make your changes / add your stuff
* Commit all your changes to your branch
* Submit a [pull request](http://help.github.com/pull-requests/)
