---
title: "Wifi Tricks and the Secret to Endless Airport Internet"
tags: ["hack", "shell", "networking"]
date: 2017-11-16T20:43:52+13:00
summary: Set a random mac address in OSX.
---

While I was in Airport, I had a time limit on my Wifi which I used up
too fast. So now all I've got on my mind is the challenge of getting
back onto the internet.

Who limits Wifi anyway? What are we? Savages?

In this case I didn't have to do much because I had a snippet of code I
took from my old workmate [Steve Purcell][1]. It's sole purpose is to
generate a random Mac address, and make that callable from the zsh
shell.

Because he never got around to publishing it, here it goes!

```sh
function randommac() {
  ruby -e 'puts ("%02x"%((rand 64)*4|2))+(0..4).inject(""){|s,x|s+":%02x"%(rand 256)}'
}
```

If you put this in your zshrc, it'll happliy generate you a mac address.
Here's how you'd call it:

```sh
randommac
> e2:f0:48:69:cc:7c
```

The only way the router can identify you is through your hardware
address. Use this command to set your wifi's mac and that should get you
around being identified. In my case, I reset my hardware address by
running:

```sh
sudo ifconfig en0 lladdr $(randommac)
```

Reconnecting got me another hit of that sweet sweet wifi! Thanks Steve!

[1]: https://www.sanityinc.com/about
