---
title: "Aggregating RSS Feeds in Scala"
date: 2017-10-04
categories: [scala,scalafx,monix,rss,programming]
tags: [scala,scalafx,monix,rss]
---
My last post talked about how one of my recurring pet projects is an [RSS][rss] reader. I consistently try and add little features that I would like, and I use it to learn new things (e.g. [Bayesian filtering][bayes]).

Since I had played around with [Monix][monix] for [Twitter][twitter] (and really liked it), I figured I'd give it a go-around with aggregating, etc. So, using [ROME][rome] to download the feeds, here's how I thought the data should flow...

* For each URL, have a periodic task to download it.
* Each time a feed is downloaded, send it to the aggregator.
* Remove all the old headlines from that feed and replace with new ones.
* Flatten all the headlines and sort them by time.

Okay, that seems reasonable. 

First, I need an aggregator to send all the feeds to. Each time one is downloaded, it'll be sent to the subject along with the URL used to download it.

```scala
val aggregator = PublishSubject[(String, SyndFeed)]()
```

Next, create the periodic tasks that will download the feeds and send them to the aggregator.

```scala
val readers = urls map { url =>
  Observable.intervalAtFixedRate(1.second, 5.minutes) foreach {
    _ => Try(download(url)) foreach (feed => aggregator onNext (url -> feed))
  }
}
```

The above assumes a `download` function, but for this post we'll just know it uses [ROME][rome] to fetch a `SyndFeed` and throws an exception if there's a failure for some reason (hence the `Try`).

The `aggregator` is now being fed `SyndFeed`s that are downloaded every 5 minutes. The problem, though, is the `aggregator` only maintains a single feed at a time. I need to pull them all together into a single container.

```scala
val feeds = aggregator.scan(Map[String, SyndFeed]())(_ + _)
```

All the `feeds` are now together in a single map. Each time a new feed is downloaded and sent to the `aggregator`, the map is updated, and the old feed is replaced with the new one.

Last, all the feed entries need to be wrapped in my own `Headline` class, flattened together, and sorted, etc.

```scala
val headlines = feeds.map(_.values.flatMap(extractHeadlines _).toList.sorted)

/**
 * Given a feed, grab all the entries and convert them to a Headline, which can be
 * used for comparison (sorting), HTML parsing, etc.
 */
def extractHeadlines(feed: SyndFeed): List[Headline] = {
  feed.getEntries.asScala.map(it => new Headline(feed, it)).toList
}
```

That's it! Not bad.

In the end, there's actually more to it than what's here. Gotta handle failure conditions, update the UI, filtering of headlines, searching, etc. But, above is 80% (and the core) of the code required to get it going.

The full app also has the following features:

* Filter feeds by source, age, or search text.
* Keep the JSON preferences and headlines in sync.
* Preview selected headlines in a side panel.
* Preview media enclosures: audio and video.
* Archive headlines.

![screenshot](https://raw.githubusercontent.com/massung/codeninja/master/_posts/images/rss-reader.png)

If you're interested, just head to [the GitHub repo][repo] and check it out! I have a bit more work to do, but I have it up and running pretty much all day. I've considered moving the code to [ScalaJS][scalajs]... we'll see. If I do, I'll be sure to post about it here, too.

[rss]:			https://en.wikipedia.org/wiki/RSS
[bayes]:		https://en.wikipedia.org/wiki/Naive_Bayes_spam_filtering
[monix]:        https://monix.io
[twitter]:		https://twitter.com
[rome]:			https://rometools.github.io/rome/
[repo]:			https://github.com/massung/scala-rss
[scalajs]:		http://www.scala-js.org
