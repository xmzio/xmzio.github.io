---
layout: post
title: Dependency injection in Cocoa view controllers
excerpt: How to implement a basic dependency injection pattern in Cocoa view controllers when using storyboards, with an approach based in protocols and optional downcasting.

---

When creating an iOS (or Mac) application, the idea of implementing a dependency injection pattern for view controllers is very attractive.

We could create custom initializers and pass objects at the time of instantiating each view controller.

However, if we want to design the UI and navigation flow in a storyboard, the custom initializers approach is not an option. View controllers instantiated from a storyboard will always use the ```init(coder aDecoder: NSCoder!)``` initializer.

Brent Simmons from Q-Branch has been blogging his progress on Vesper for Mac, and some of his recent posts describe his line of thinking about how to *"pass things around"* ([here](http://inessential.com/2014/07/05/vesper_mac_diary_5_storyboards_and_pa), [here](http://inessential.com/2014/07/05/vesper_mac_diary_6_prepareforsegue_bu) and [here](http://inessential.com/2014/07/05/vesper_mac_diary_8_figured_out_how_to)) which inspired this article.


This is how I have accomplished it:

First create a protocol specifying properties for the objects you want to pass around:
{% highlight JavaScript %}
import CoreData

@objc protocol ViewControllerWithContext {
	var context: NSManagedObjectContext! { get set }
}
{% endhighlight %}

And make your view controllers conform to the new protocol:
{% highlight JavaScript %}
import UIKit
import CoreData

class ActivitiesViewController: UITableViewController, ViewControllerWithContext {

    var context: NSManagedObjectContext!
	...
{% endhighlight %}


Note the _@objc_ attribute in the protocol declaration. The Swift Programming Languaje guide states that:

> Even if you are not interoperating with Objective-C, you need to mark your protocols 
> with the @objc attribute if you want to be able to check for protocol conformance. 


Now you can inject the object (the *context* property, in this case) when preparing the segue:

{% highlight JavaScript %}
override func prepareForSegue(segue: UIStoryboardSegue!, sender: AnyObject!)  {

    if let destinationViewController = segue?.destinationViewController as? ViewControllerWithContext {

        destinationViewController.context = self.context
    }
}
{% endhighlight %}



For the application's initial view controller, the only place where we can pass objects is in the ```application:didFinishLaunchingWithOptions:``` method, in the AppDelegate.

Here I added an extra type checking, because sometimes I swap my initial view controller with a Navigation Controller in the storyboard.

I would probably remove the type checking block before shipping the app.


{% highlight JavaScript %}
func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: NSDictionary?) -> Bool {

    var candidateViewControllerWithContext: ViewControllerWithContext?

    // If the initial view controller is a Navigation Controller...
    if let navigationController = self.window!.rootViewController as? UINavigationController {

        candidateViewControllerWithContext = navigationController.viewControllers[0] as? ViewControllerWithContext
    }

    // If is NOT a Navigation Controller...
    else {

        candidateViewControllerWithContext = self.window!.rootViewController as? ViewControllerWithContext
    }

	// If we got a ViewControllerWithContext...
    if candidateViewControllerWithContext {

        candidateViewControllerWithContext!.context = self.context
    }

    return true
}
{% endhighlight %}

This code may not look clean (nesting code in conditional type checking is not always a good practice) but this is the easiest way to implement a dependency injection pattern in Cocoa view controllers, and continue enjoying the benefits of the flexibility that storyboards provide.
