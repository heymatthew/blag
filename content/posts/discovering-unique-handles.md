---
date: "2016-09-23T16:51:31+13:00"
title: "Discovering Unique Handles"
summary: Brute forcing a username across socail networks with Ruby.
tags:
  - script
  - ruby
  - irc
---

For years I used the name "Daemons Advocate". I don't know why, but at some stage I stopped liking it. This got tricky, anytime I get to know a name I start to dislike it.

Picking a handle is too hard. And everything I came up with is taken by some other schmo on the internets because if you're looking for a good short name in 2016, you're way late to the party.

Generating a randomly pronouncable name for servers is something [Tom][2] (a mate of mine) dreamt up and I never got sick of those names.

So why not go further and use that same technique for usernames? Lets pwgen a short name out of garbage and attach an identity to something pronouncable!

What do we need?

1. A way to generate handles, ala pwgen or counting upwards
2. A decent way to check if they're taken, ala twatter 404 headers
3. Some way to generate some limited set of passwords to geez at

I've been meaning to try make some kind of enumerable for a while, so I'm going to try hone that ruby. If you're going to play along at home you're gonna need `pwgen`, `ruby` and `vim` binaries. The last one is not required, but highly recommended ;-).

```ruby
HandleGenerator = Struct.new(:name_length) do
  def next
    return enum_for(:next) unless block_given?
    loop { yield pwgen_name }
  end

  private

  def pwgen_name
    `pwgen -0A #{name_length}`.chomp
  end
end
```

Awesome! Now we can pick names!

```ruby
name_length = 6
name_enumerator = HandleGenerator.new(name_length).next
10.times { puts name_enumerator.next }
```

But before I continue down that path, I'm curious about the shorter handles as these would be hot property. Fun facts:

* Twatter gives 404s for some handles but calls them "taken" if you go to register them. So a 404 gives a possible name, but not a fo' sure name.
* There are 26^2 = 676 handles with 2 chars, and these 15 appear untaken but seem to be reserved: ie, is, wi, hv, he, pb, pu, rp, zb, vy, co, jz, gl, fg and uk
* There are 26^3 = 17,576 handles with 3 chars, and 461 appear to be untaken

Hitting their API will give me assurance but it's rate limited to 100 hits an hour-- or something. I don't care to read their 1155 word [api essay][1] because it's sooooo.

So brute force 404s, then take the shortlist to the api?

Woo!

To get around rate limiting, I'm going to try hit public pages that'd usually return 404 if the handle isn't taken. I'll let you use your imagination, or read [that essay][1] if you want to keep going with that.

Here's what I ended up with...

```ruby
require 'net/http'

HandleAvailable = Struct.new(:name) do
  def call
    services.shuffle.all? do |service|
      response_code(:from => service) == 404
    end
  end

  private

  def response_code(from:)
    Net::HTTP.get_response(from).code.to_i
  end

  def services
    [
      URI.parse("https://twitter.com/#{name}"),
      URI.parse("https://github.com/#{name}"),
      URI.parse("https://disqus.com/by/#{name}/"),
      URI.parse("https://www.youtube.com/user/#{name}"),
      URI.parse("http://#{name}.tumblr.com/"),
    ]
  end
end

NextAvailableHandle = Struct.new(:name_generator) do
  def next
    return enum_for(:next) unless block_given?
    loop do
      yield next_available_name
    end
  end

  private

  def next_available_name
    loop do
      name = name_generator.next
      return name if HandleAvailable.new(name).call
    end
  end
end

name_enumerator = HandleGenerator.new(name_length).next
search_enumerator = NextAvailableHandle.new(name_enumerator).next

# find me 10 usernames that might be untaken
10.times do
  puts search_enumerator.next
end
```

Now what you end up with does need a bit of human filtering based on taste, but they're all pronouncable. To sample-- eitohs, ohphah, and voatee are all names that came out of running this.

All stories deserve a happy ending. I found that I liked the name `@wohyah` and I don't think that's half bad!

[1]: https://dev.twitter.com/rest/public/rate-limiting
[2]: https://twitter.com/tveastman
