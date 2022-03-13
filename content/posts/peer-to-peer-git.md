+++
date = "2017-03-31T17:37:49+13:00"
title = "Peer to Peer Git"
+++

Sometimes your central git repository goes down, but you need to share code. That's a bummer, you can commit and wait, but you've likely heard that you shouldn't need to rely on a central repo.

So you've got a few options here:

1. Mirror your repo someplace else and switch upstreams during outages,
2. email your stuff as patches using [git am][2],
3. setup a small read only local git server and pull between neighbours,
4. or use some kind of highly available decentralised storage as an upstream like [ipfs git][4].

I feel the easiest thing to do is just to run that read only git option #3. And it's not that hard either, you just need a git alias to run in a shel, and your neighbour to configure and pull from you.

# Setting up your repo

Firstly, create a 'serve' alias for your user so you can share your work...

```bash
git config --global alias.serve "daemon --verbose --export-all --base-path=.git --reuseaddr --strict-paths .git/"
```

Now navigate to your repo and run it in the foreground...

```bash
cd $MY_REPO
git serve
```

This should say "Ready to rumble" followed by log when people connect to you. When you want to shut it down, just it "Control C" and it'll kill the git server.

# Connecting to your neighbour

Assuming you've setup your server, you're now going to want to know your IP address or hostname so you can tell other people who to connect to.

To do this on OSX or Mac (sorry Windows), run ipconfig from a shell and pick your IP out. Usually this is a 192.168.x.x or a 10.x.x.x style network. Usually on Linux that'll be next to a device called wlan0 or eth0, and on OSX en0. If you have a machine name, that can sometimes work to! Just run `hostname` and note the output.

From henceforth, I'm going to write `$YOUR_BOX` to represent this thing we just looked up. Just sub in whatever you got from the above.

Righty ho! Now that you have that, lets get your neighbour connected to `$YOUR_BOX`! Lets add you to your neighbours remotes...

```bash
git remote add neighbour git://$YOUR_BOX/
git fetch neighbour
```

Boom! Now lets set that branch as an upstream!

```bash
git checkout master
git branch --set-upstream neighbour
```

If you run a server, and so does your neighbour, the you can pull in each others changes and keep working.

[2]: https://git-scm.com/docs/git-am
[4]: https://github.com/larsks/git-remote-ipfs
