---
layout: single
title: _Lockdep, step back to see it wholly._
date: 2018-12-05 21:40:08 +0900
categories: linux
---
You would often been confronted with various kinds of lockdep notations, or even have to choose an appropriate one to use, when you would have to dive into linux kernel for some reason. This post presents its overview, which I think is rarely illustrated as a whole. Note that I assume you have already read the [kernel doc](https://www.kernel.org/doc/Documentation/locking/lockdep-design.txt). Obviously it's indispensable material and the core concept is clearly explained.

The first pic is its usage pattern. You can click it to zoom in:

[![lockdep use](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0002-lockdep-use-overview.png){:class="img-responsive"}](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0002-lockdep-use-overview.png)

and the next one depicts the almost all relating data structure:

[![lockdep use](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0003-lockdep-data-structure.png){:class="img-responsive"}](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0003-lockdep-data-structure.png)

Now that you've grasped the whole picture of lockdep, I would like to show you what I think is one of the most interesting reality of how it's been used, i.e., reality sucks. The scene is where the lockdep has to be fooled by aio.

[![lockdep use](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0006-lockdep-fooled-by-aio.png){:class="img-responsive"}](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0006-lockdep-fooled-by-aio.png)

The point is that:
- (A) To protect against freezing when aio_completion, we postpone [`__sb_end_write`](https://elixir.bootlin.com/linux/latest/ident/__sb_end_write) until [`aio_complete`](https://elixir.bootlin.com/linux/latest/ident/aio_complete).
  - However, that would introduce situation where [`held_lock`](https://elixir.bootlin.com/linux/latest/ident/held_lock) exists being unreleased when returning to userspace.
    - see: [`lockdep_sys_exit`](https://elixir.bootlin.com/linux/latest/ident/lockdep_sys_exit)
- (B) So, just to avoid lockdep warning, we negate. Yes, we fool lockdep.

I tried in the past to tackle this so as to remove the hack but I couldn't offer any good solution.

I hope this post helps someone.
