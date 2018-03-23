+++
title =  "You should not bind port 3306 to your host"
date = 2018-03-23T00:16:20+01:00
tags = ["docker", "security"]
featured_image = ""
description = ""
+++

In this post, I would like to warn you about the risk of binding your data stores (databases, caches, event stores, ...) container ports to your local machine. I assume this is a quite common use case for developers using `docker` and/or `docker-compose`. But, unfortunately, this is mostly wrong and might become a serious security risk sooner or later.

## But why?

Sometimes, for whatever reason, developers using `docker-compose` want to access their data store from their local machine. And most of the time, it ends up with something like that:

```bash
docker run -d -p 3306:3306
# or
docker run -d -P mysql
```

```yaml
services:
  db:
    image: mysql
    ports:
      - 3306:3306
```

If you ever worked for quite some time on the same project, or get assigned on some nasty bugs, you might have ended up **loading a dump of the production database in your container**. You know what?

Weeell, **this is wrong!** *Why?* We need to take a look at two pages of Docker documentation first. More precisely, to the the end of the page "[Bind container ports to host](https://docs.docker.com/v17.09/engine/userguide/networking/default_network/binding/)" (4th paragraph before the end):

> You can see that Docker has exposed these container ports on 0.0.0.0, the wildcard IP address that will match any possible incoming port on the host machine.

As explained in this sentence, **whenever you publish a port and don't specify an IP address, Docker binds to `0.0.0.0`**, or said in another way: it allows traffic to come in from anywhere, with no restriction whatsoever.

**This is especially wrong** if you work outside of your office, like **if you bring your laptop to conferences, or you do remote work in coworking spaces, cafes, and places alike**. Moreover, you might work for high stakes clients, and there's now `GDPR` coming in. So, in that setup, you just let potential faceless anonymous hackers suck your database up. What a shame!

## Let's fix up!

But hey, there's a good news: now you're aware of that shitty situation, you can do the right things to fix it :-)

Yeah, I know you've always been like *I'm using a fancy Linux distro, so what could go wrong? Firewall? Antivirus? Nope, don't need that, this is for Windows guys and girls...* But really, **your OS won't protect you against yourself leaking data on public networks**. So the first thing to do is: **use a firewall.**

Depending of your OS (and disto, if you're a Linux aficionado) it may vary a lot, but the general idea is to block any incoming traffic while preserving outbound traffic.

Why do so? Because, there's nothing like a firewall that could prevent any actual and future data leak because of unchecked data store exposures. You also need to understand (or remember) how a security chain works: it is as strong as its weakest link. Hence the best is always multi-layered security.

## The easy fix

Yeah, at that point of article, you might have thought about another way to fix the problem: **bind container ports to the local IP address, `127.0.0.1`, rather than `0.0.0.0`**.

You might even have read how to do at the end of the 4th paragraph before the end of the page, in the 2nd link above:

> When you invoke docker run you can use either -p IP:host_port:container_port or -p IP::port to specify the external interface for one particular binding.

*Great, but this is only for `docker`*, you might say. For `docker-compose`, this is the same in fact: **you have to specify the IP address to bind to, in front of the pair of ports**.

```bash
# This is good:
docker run -d -p 127.0.0.1:3306:3306
# This should never be done:
docker run -d -P mysql
```

```yaml
services:
  db:
    image: mysql
    ports:
      - 127.0.0.1:3306:3306
```

*And voila!*
