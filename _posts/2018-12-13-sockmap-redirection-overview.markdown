---
layout: single
title: _Sockmap redirection overview_
date: 2018-12-13 06:12 +0900
categories: linux
---
How [sockmap](https://lwn.net/Articles/731133/) redirection are set up and executed seems rarely depicted so I've done it. Cilium is becoming more popular these days, which utilises sockmap (more precisely, sockhash) as an optimisation.

Please beware that:
- the "T" and "R" of the [T1]~[T3]/[R1]~[R3] written white in red is "TX" and "RX" respectively, and the [T1] and [R1] in sk_psock->progs are the same for the same socket.
- As noted just above, this figure contains both SEND-side and RECV-side redirections so it looks somewhat confusing.

[![0004-sockmap-overview.png](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0004-sockmap-overview.png){:class="img-responsive"}](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0004-sockmap-overview.png)
