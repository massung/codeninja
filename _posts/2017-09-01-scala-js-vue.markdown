---
title: "ScalaJS Facade for Vue.js"
date: 2017-09-01
categories: [scala,js,facade,vue,programming]
tags: [scala,js,facade,vue,reactive]
---
I had just finished up my [ScalaJS G8 Skeleton][skeleton], and thought I'd take a little time out from the daily grind and play around with reactive frameworks. And what better first app than the "hello world" of web apps: a TODO list?

The last reactive framework I toyed around with was [Angular][angular] 1.0 (right as it was transitioning to 2.0) quite a long time ago, and while I liked some of the ideas (`ng-` tags), but couldn't stand that ["hammer factory hell"][factory] that it had going on.

I started looking around at the other options out there, and - to be honest - I really didn't like much of what I saw. I'm an old-school programmer, and while production code and libraries always grow as they solve more and more problems, they should still be able to introduce new-comers to the basics with a few lines of code and slowly build from there.

I then came across [Vue.js][vue] and was very pleasantly surprised. It had what I enjoyed from [Angular][angular], the entire API fit on a single page, and the [Getting Started][start] section was a pleasant, gentle, step-by-step introduction to everything one needed to know to get up and running quickly.

Next was to see if there was already a [Vue][vue] facade for [ScalaJS][scalajs], and [there was](https://github.com/fancellu/scalajs-vue). However, it was very bare-bones (not the entire API), and not very type safe. Admittedly, [Vue][vue] functions accept variant types: DOM elements, element selector strings, arrays of both, and even functions. So, it can be difficult to steer clear of `js.Any`. But, I hadn't made a facade before, [Vue][vue] was small enough, and I thought others might get some use out of it as well.

Well, a couple days later and my [ScalaJS Vue Facade][facade] is up and ready to be used from [JitPack][jitpack]!

The entire [Vue.js API][api] should be supported with a good amount of type safety. As I learn more of the [ScalaJS][scalajs] facade tricks-of-the-trade, I'll add more type safety as well.

As always, if you find something wrong - or have a better way of doing something in the facade - please, open an issue and tell me about it!

[skeleton]:     http://codeninja.blog/2017/scala-js-skeleton
[scalajs]:      http://www.scala-js.org
[sbt]:          http://www.scala-sbt.org
[factory]:      http://discuss.joelonsoftware.com/default.asp?joel.3.219431.12
[vue]:          https://vuejs.org
[angular]:      https://angularjs.org
[start]:        https://vuejs.org/v2/guide/#Getting-Started
[facade]:       https://github.com/massung/scala-js-vue
[jitpack]:      https://jitpack.io/#blog.codeninja/scala-js-vue
[api]:          https://vuejs.org/v2/api
