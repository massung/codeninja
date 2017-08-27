---
title: "ScalaJS App Skeleton"
date: 2017-08-24
categories: [scala,js,programming]
tags: [scala,js,sbt,electron,express,skeleton]
---
I've had a lot of fun with [Scala][scala] over the past couple years doing little side things here and there. When I want to do some FP (and still be productive), Scala is my goto language.

In the past I've tried to pick up [ScalaJS][scalajs], but it was always just a bit of an annoyance to get all the tooling together, and I was anxiously waiting for [Scala][scala] 2.12 features. Well, not only is 2.12 out, but [ScalaJS][scalajs] is very close to 1.0!

So, I decided to just sit down over the past couple days and crank out a [skeleton][zip] app that can be downloaded, easily modified, and instantly run out of the box (heavy on the comments for newcomers).

The skeleton uses [SBT][sbt] to compile and build the app. It's pre-setup for use with [Electron][electron] (for desktop apps) or [Express][express]. It's also pre-setup for use with [VSCode][vscode], as it has 3 launch configurations: run, debug, and serve.

![skeleton](https://raw.githubusercontent.com/massung/codeninja/master/_posts/images/scala-js-skeleton.png)
[https://github.com/massung/scala-js-skeleton/](https://github.com/massung/scala-js-skeleton/)

I hope this gets a few more people using [ScalaJS][scalajs] and/or [Scala][scala]!

If you find something wrong - or a better way of doing something in the skeleton - please, open an issue and tell me about it.

[scala]:        http://www.scala.org
[scalajs]:      http://www.scala-js.org
[nodejs]:       https://nodejs.org
[electron]:     https://electron.atom.io
[express]:      http://expressjs.com
[sbt]:          http://www.scala-sbt.org
[vscode]:       https://code.visualstudio.com
[zip]:          https://github.com/massung/scala-js-skeleton/archive/master.zip
