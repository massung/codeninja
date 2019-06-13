---
title: "Exploring Nim"
date: 2019-06-12
categories: [programming,nim,hn]
tags: [programming,nim,hn]
---
There are a few new programming languages right now vying for the new title of "C, but better." One in particular that I've been watching for a while now is [Nim][nim]. It has a couple rough edges, but now that it's getting close to 1.0, I wanted to give it a closer look and actually try implementing something with it.

## The Problem

Whenever I sit down to learn a new programming language, I make something that I want and will actually use. This forces me to do all the little things that really make an app stand out for me and not cut corners. Over the years, probably my favorite little app to make has generally been either some kind of emulator for an esoteric CPU/console or news reader/aggregator (e.g. RSS, [Hacker News][hn]).

I settled on [Hacker News][hn], but I wanted it to be something that could just be running to the side in [tmux][tmux] on my Linux machine and run on Windows as well. It's also a great little test for any new language. 

Features that I wanted it to have:

* Downloading stories in parallel
* ANSI terminal controls
* REPL-like behavior with command parsing
* Sorting, searching, and browser launching

In many languages, many of these features should be dead simple to implement, but it's also amazing how many languages tank miserably or require lots of 3rd-party libraries and extra code. Maybe it's me getting old, but I prefer the languages I use today to essentially be "batteries included" languages. I don't want to have to roll my own HTTP client library or JSON parser before I can start working on the _real_ problem I want to solve.

## Diving In

After getting acquainted with the syntax and subtle differences from other languages (not long, [Nim][nim] is quick to pick up), the first thing to figure out was the HTTP client and JSON parsing.

[Nim][nim] is definitely a "batteries included" language in the same way [Go][go] is (although I - overall - tend to dislike [Go][go] for a myriad of reasons). Downloading and parsing stories from [Hacker News][hn] turns out to be quite simple with [Nim][nim]:

```nim
import httpclient
import json
import options
import strformat

type
  Story* = JsonNode

const api = "https://hacker-news.firebaseio.com/v0"

proc hnGet*(path: string): JsonNode =
  let body = newHttpClient().getContent(fmt"{api}/{path}.json")
  return parseJson(body)

proc hnGetTopStories*(): seq[int64] =
  return to(hnGet("topstories"), seq[int64])

proc hnGetStory*(id: int64): Option[Story] =
  let json = hnGet(fmt"item/{id}")

  if json.kind == JNull:
    return none[Story]()
  else:
    return some(json.Story)
```

There, a very simple wrapper for downloading all the available top stories from [Hacker News][hn] and parsing them as JSON.

_NOTE: The [Hacker News API][api] has some quirks about it that make it difficult to parse stories as directly as one would hope. If a story has been deleted, the API can return `null` or an empty JSON object `{}`, not all fields of a story are necessarily filled out, and even when they are, they can often be the default value (e.g. a `url` can be a missing key or an empty string `""`)._

## Going Concurrent

One of the annoyances of the [Hacker News API][api] is that it doesn't just straight-up return a list of stories as opposed to a list of their IDs (up to 500!). It is then up to you to download them _individually_. And, they are not in any discernable order, so if you want to display them in page-rank order, you can't just download the first few IDs in the list; you must to download them all, filter, and sort appropriately.

I needed the above code to actually be asynchonous: once I had a list of IDs, I needed to spin up to 500 "threads" (green threads, coroutines, whatever the language provided), download the stories in parallel, and collect the results together again.

In [NodeJS][node] this would be done with promises, [Go][go] would use goroutines and channels. In [Nim][nim] this is done with futures and the async/await pattern:

```nim
proc hnGet*(path: string): Future[JsonNode] {.async.} =
  let resp = newAsyncHttpClient().getContent(fmt"{api}/{path}.json")
  return parseJson(await resp)

proc hnGetTopStories*(): Future[seq[int64]] {.async.} =
  return to(await hnGet("topstories"), seq[int64])

proc hnGetStory*(id: int64): Future[Option[Story]] {.async.} =
  let json = await hnGet(fmt"item/{id}")

  if json.kind == JNull:
    return none[Story]()
  else:
    return some(json.Story)
```

Very little changes were required, but now the code could run asynchronously, and I could write a simple function to download a sequence of stories in parallel quite easily:

```nim
proc hnGetStories*(ids: seq[int64]): Future[seq[Option[Story]]] {.async.} =
  let futures = ids.map(hnGetStory)
  let stories = await all(futures)

  return stories
```

I won't dive into the terminal code or user-input. That's pretty "meh" in any language. But, there were a few pieces there that really stood out nicely to me:

* Batteries included support for the terminal, ANSI colors, etc.
* Create compile-time platform support and trivial C interface for the tiny bit of work I needed to do in order to get unicode and colors working on Windows.
* Enough built-in reflection to allow parsing enumerations.
* Closure iterators. 

The end-code - available on [GitHub][repo] - does a bit more for tracking download progress, etc. but that's essentially all there is to it!

## The End Result

Here is a little screencast of the final app in action:

![Hacker News](https://raw.githubusercontent.com/massung/hn-reader/master/screenshot.gif)

On Windows, the executable is just barely over 650 KB in size and it runs very well; it takes ~3-5 seconds on average to download 500 stories.

For comparison, my version of this in [Rust][rust] is over 5 MB compiled and takes ~6-8 seconds on average to download the same set of stories.

_NOTE: Both were built in release mode with no other optimization flags. Obviously this is a trivial comparison, but I've been playing with both languages as my "alternative to C"._

## What I Like

I'll say that I haven't had this much fun whipping up a little toy program in a new language in quite some time. Here's a simple list of things that stood out to me positively:

**Batteries Included**

I noted this above: [Nim][nim] is a "batteries included" language. Any "modern" language should include support for HTTP, JSON, threading, unicode, regular expressions, interfacing with C, etc. But, [Nim][nim] does so much more:

* Parsing: JSON, XML, INI, SQL, PEG, CSV, CLI, RST, ...
* Asynchronous: streams, futures, async/await, channels, thread pools, ...
* Internet: CGI, HTTP, SMTP, sockets, OpenSSL, ...
* Crypto: MD5, SHA, BASE64, ...
* Strings: Regular expressions, unicode, ropes, ...
* Databases: PostgreSQL, MySQL, SQLite, ODBC

And there's plenty more out of the box: check out the [standard library][stdlib].

**Pragmas**

I must admit that over the past few years, whenever I would stop to look at [Nim][nim], the pragmas always muddled the code to me and took away from - what otherwise would be - simple, clearn, Python aesthetics. But, once I started actually coding with it, I found them to be a well thought out addition. Instead of adding extraneous keywords, the pragmas allowed the ability to relay lots of (optional) information to the compiler if necessary. And they could be used to enforce requirements at compile time.

Initially, the only insight I had into them was the `{.async.}` pragma (which could be a keyword), and the FFI pragmas for exposing C functions:

```nim
proc setConsoleOutputCP(page: int): WINBOOL
  {.stdcall, dynlib: "kernel32", importc: "SetConsoleOutputCP".}
```

But, it ends up there are far more. My "favorite" (can one have a favorite pragma?) so far is `{.gcsafe.}`. Being able to mark a function as not accessing any global memory that can be garbage collected.

Most of the time, I think of pragmas in [Nim][nim] as being like types in a type-inferred language: you can ignore them except when it would be really nice to tell the compiler _exactly_ what you want, and then they are incredibly useful.

_Side note on pragmas: what I like most about them is that they do take the place of many keywords. In the future, this will make the language incredibly flexible since they are just hints for the compiler. Adding/removing keywords from a language (after it's reached 1.0) is incredibly painful. Adding/removing pragmas is much easier._

**Uniform Calling Syntax**

Some people love it, others hate it. I love it. For those who don't know what it is:

```nim
proc add_10(x: int): int = x + 10

echo 5.add_10()
echo add_10(5)
```

I love it because I've always disliked putting methods inside a class/object. Objects should be data, what can be done to/with that data is far-reaching, and not just what the original author intended. UCS allows free functions to accomplish the same goals as extension methods, but without the added syntax.

This also enables various functions to be imported as needed. The `seq` type doesn't come pre-loaded with `map`, `filter`, `zip`, etc. bloating your final executable. Instead, just `import sequtils` if needed, and then you get them.

**Identifier Equality + UCS**

See: [Identifier Equality][ident].

Every programmer has their own dogma around identifiers: snake_case, camelCase, alllowercase, etc. [Nim][nim] allows for library authors to write the code their way, and for users to use it their way:

```nim
proc do_something_cool(msg: string) = echo msg

do_something_cool("hello, world!")
do_SomethingCool("hello, world!")

"hello, world!".doSomethingCool
```

It's amazing how this "just works" and doesn't seem to lead to any problems. It's really quite nice.

**It's The Little Things**

There are also many "little" things that end up making a big difference:

* Closure iterators;
* Compile-time conditionals;
* Using `*` to denote public access;
* Using `$` as a `tostring` operator;
* String concatenation with `&` (`+` is commutative!);
* Ranges and subranges;
* Arrays indices not having to start at `0`;
* How enumerations were implemented;
* Use of `high` and `low`;
* No `main` function;
* Templates;
* and more...

## Things I Dislike

No language is perfect for all programmers. We each have our little "I wish..." thoughts with every language. Here are some of mine frustrations:

* Verbose syntax for anonymous procs;
* Too much reliance on exceptions when there is an `Option` type;
* Confusion around when `return` is needed and not;
* Spawned tasks are confusing (see: `FlowVar`);
* Over-use of operators (I'm against overloading operators);

I'm sure there will be more with time...

## Verdict

In the end, the final verdict on a language (to me) is how productive I am with it, and whether I had fun using it. Did I spend most of my time reinventing the wheel? Was I fighting syntax and the compiler (staring at you, [Rust][rust])?

I enjoyed programming in [Nim][nim] very much. In fact, this is the first OSS language I've actually gone and opened issues for with code to help fix a couple bugs. And I can see myself contributing more to it as time goes on.

Probably the best compliment I can give [Nim][nim]: I'm already musing over what my next project with it will be!

# fin.

[nim]: https://nim-lang.org/
[hn]: https://news.ycombinator.com/
[tmux]: https://github.com/tmux/tmux/wiki
[go]: https://golang.org/
[api]: https://github.com/HackerNews/API
[node]: https://nodejs.org/
[repo]: https://github.com/massung/hn-reader/
[js]: https://nim-lang.org/features.html
[stdlib]: https://nim-lang.org/docs/lib.html
[rust]: https://www.rust-lang.org/
[ident]: https://nim-lang.org/docs/manual.html#lexical-analysis-identifier-equality