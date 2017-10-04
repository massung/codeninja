---
title: "Twitter Client in Scala"
date: 2017-10-04
categories: [scala,scalafx,monix,twitter,programming]
tags: [scala,scalafx,monix,twitter]
---
One of my favorite projects when learning a new language is an [RSS][rss] reader. It tends to hit many of the major "can I get things done?" parts of a new language that interest me:

* Net libraries
* Threading
* Parsing
* Error handling
* Method dispatch
* Multi-source aggregation
* UI/reactive

And as easy as it sounds, there's quite a lot that can go into such a simple project. And this gets even more "fun" when you consider adding in UI (whether it's web or desktop).

I did this quite a while ago for myself in [Scala][scala] when first learning it, but over the course of time, [RSS][rss] has been falling out of favor. There have been lots of side services that have popped up attempting to replace it, but I wanted to build a new app for myself trying something different.

I've been pretty much behind-the-times when it comes to [Twitter][twitter]. I have an account, but rarely ever used it. But, I noticed that [Twitter][twitter] has support for lists, and I thought to myself, "hmm... major news outlets tweet, maybe I'll just use [Twitter][twitter] as my news aggregator?" And, so, off I went to start putting together a [Twitter][twitter] client using [Scala][scala].

Now, it'd been a while since I put together a desktop app in [Scala][scala], and since then [ScalaFX][scalafx] has come along nicely, and when combined with [Monix][monix], say "hello" to some snappy, easy-to-build, functional, reactive user interfaces for the desktop!

```scala
val verifiedUser = PublishSubject[Option[twitter4j.User]]()

// When the user changes, download all the users's list subscriptions.
val userLists = verifiedUser flatMap {
	case None       => Observable.now(List[twitter4j.UserList]())
	case Some(user) => Observable.fromTask(getUserLists(user.getId))
}

// When the user changes, download all the saved searches.
val savedSearches = verifiedUser flatMap {
	case None       => Observable.now(List[twitter4j.SavesSearch]())
	case Some(user) => Observable.fromTask(getSavedSearches(user.getId))
}

// When the lists/searches have been downloaded, update the menu items.
userLists foreach { lists => /* populate menu */ }
savedSearches foreach { searches => /* populate menu */ }

// When the user authenticates, toggle the interface to the timeline view.
verifiedUser foreach {
	case Some(_) => pane.center = timelineView
	case None    => pane.center = loginView
}
```

The above code is a bit simplified, but should get the idea across. Everything is just a series of background tasks that run and - as they complete (or fail!) - the interface will react accordingly.

If you haven't yet had a chance to try [Monix][monix] (or [ScalaFX][scalafx]) in your project, I recommend coming up with an excuse to give them both a try... together if possible!

![screenshot](https://raw.githubusercontent.com/massung/codeninja/master/_posts/images/scala-twitter-client.png)

*RIP Tom Petty!*

I still have a few more features to add before I'd consider it "RC-1", but all the basics are in place and I pretty much have it running daily on my work machine and at home. I hope to make the app available at some point in the near future and possibly the sources as well.

[rss]:			https://en.wikipedia.org/wiki/RSS
[scala]:      	http://www.scala-lang.org
[scalafx]:      http://www.scalafx.org
[monix]:        https://monix.io
[twitter]:		https://twitter.com
[twitter4j]:    http://twitter4j.org
