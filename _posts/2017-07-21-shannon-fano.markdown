---
title: "Shannon-Fano coding in Go"
date: 2017-07-21
categories: [go,shannon,fano]
tags: [go,shannon,fano]
---
For another project I'm working on I needed to [Shannon-Fano][1] encode text strings into bit vectors. So, I cranked out a [Go][2] package real quick and thought others might find it useful as well.

[http://github.com/massung/go-shannon](http://github.com/massung/go-shannon)

Aside from - well - encoding, I wanted to be sure the table could be serialized using [encoding/gob][3], that way it would be possible to use the encoding (and decoding) via files and even over the network. There's an example of doing this in the [README][4].

Cheers! I hope you find it useful.

[1]: https://en.wikipedia.org/wiki/Shannon–Fano_coding
[2]: https://golang.org
[3]: https://golang.org/pkg/encoding/gob/
[4]: https://github.com/massung/go-shannon/blob/master/README.md