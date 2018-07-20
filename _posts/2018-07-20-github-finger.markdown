---
title: "GitHub Finger"
date: 2018-07-20
categories: [github,finger,programming,scala]
tags: [github,finger,programming,scala,http]
---
I missed the [finger][man] [protocol][protocol]. 

It was such a simple, concise method for tech-savvy individuals to share what they were working on with the world and also discover what their friends and colleagues were contributing to.

I wanted that back. [^1]

## Creating a New Finger

The idea was to create a simple REST API that could be used to lookup a person and get "plan" information about them in text, markdown, JSON, or even HTML. I wanted something that could be embeded in documents, blogs, or even linked to in a tweet. I also was hoping to piggy-back off something that not only already existed, but was ubiquitous...

[GitHub][github] is where the (programming) world works today, and its open [API][api] provides exactly what I need to create a new, modern version of this useful tool that would provide: 

* [biographical][bio] information
* [README][readme] to talk about themselves
* [issues][issues] and tasks
* project contributions and activity

So, that's what I did... [^2]

{% include finger.html %}

You can use this to look up the information on any [GitHub][github] user. If that user _also_ has a `.plan` repository, then the repository's README and issues (as a list of what that user is currently working on) will also be shown.

## Usage

Head over to [http://finger.codeninja.blog](http://finger.codeninja.blog/) to learn how you can link a plan and even embed it on your own web page/blog. It's easy!

{% include plan.html %}

# fin.

[^1]: Obviously `fingerd` and the [finger][man] command still exist, but they are obscure and rarely used (by anyone I know) any more.

[^2]: For those who care about such things, this was created with [Scala][scala].

[protocol]: https://en.wikipedia.org/wiki/Finger_protocol
[man]: https://linux.die.net/man/1/finger
[github]: https://github.com/
[api]: https://developer.github.com/v3/?
[scala]: https://scala-lang.org/
[bio]: https://help.github.com/articles/adding-a-bio-to-your-profile/
[readme]: https://help.github.com/articles/about-readmes/
[issues]: https://help.github.com/articles/about-issues/
[plan]: http://finger.codeninja.blog/?q=massung
[create]: https://github.com/new
[scala]: https://scala-lang.org/