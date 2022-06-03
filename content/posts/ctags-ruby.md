---
title: "CTags in Ruby: Make code feel like a wiki"
date: "2017-02-21T08:49:22+13:00"
categories: ["Vim", "Ctags", "Ruby", "Git"]
summary: Hook up jump to definition for Ruby in Vim.
---

In a browser you markup for links. A wiki builds on this behavior to describes topics, linking nouns to pages in context that link to other nown pages.

Browsing a wiki allows you to dive into information and build up your mental model of a subject area-- which is kind of addictive. Why can't we do the same thing in our code?

Code has syntax which are just links between files. We often do this linking in our heads, but our editor should be helping you out more-- at some stage there's a black box that you've never stepped into because it's outside of your project.

There have been attempts to make code behave like links through static and dynamic analysis. Here are some of the projects on my radar that pull this off:

1. [Resource Navigation][1.1] integrated throughtout the Eclipse IDE
2. [Ctags][2.1] integration for vim
3. [Go Guru][3.1] integration via the [vim-go][3.2] plugin
4. [Sourcegraph][4.1] integartion via the [vim-sourcegraph][4.2] plugin

If you're hacking on an opensource project, then sourcegraph is the natural choice at the moment as it gives you access to community and examples you may not have even checked out.

However I'm a Rubyist on a closed source thing at work, and I don't think I can convince my managers just yet to shell out for a sourcegraph subscription. This leaves me with ctags.

Ctags create a "tags" file which Vim knows how to navigate out of the box with `Ctrl + ]`. Those tags are a snapshot of static analysis mapping the links between files in your project.

If you don't want to mess up your project files though, and you have [vim-fugitive][5], then I recommend you specify your tags file location in `.git/tags`.

Initially I found plugins that ran this command on every file write. However this quickly became unweildy as it builds a massive head of background procs that turn your responsive computer into a slow hulking mass of rage.

A better way to think about this is "when you think code is ready enough to share". This happens for me when I run git commands, specifically `git commit` and `git checkout`.

You can run actions post by adding executable scripts to `.git/hooks/` with special names. There's a great overview on [githooks.com][6], but for this exersise I'm only interested in `post-commit`, `post-checkout` and `post-rewrite`. There are others, but these are the ones I'm starting with.

First, lets make all these scripts the same
```bash
ln -sf .git/hooks/{post-commit,post-checkout}
ln -sf .git/hooks/{post-commit,post-rewrite}
ls -al .git/hooks
```

Now lets hack together `.git/hooks/post-commit`:
```ruby
#!/usr/bin/env ruby

require 'time'

class TagBuilder
  CTAGS_TMP = ".git/ctags.tmp"
  CTAGS_FILE = ".git/ctags" 

  def initalize
    if !ctags?
      puts "ctags not present on system"
      exit(0)
    end

    if !gemfile?
      puts "cowardly exiting a project without a gemfile"
      exit(0)
    end
  end

  def call
    create_tmp_tags_for_project and make_tmp_tags_current
  end

  private

  def create_tmp_tags_for_project
    puts "building #{CTAGS_TMP}"
    system("ctags -R -f #{CTAGS_TMP} #{ctags_paths}")
  end

  def make_tmp_tags_current
    puts "updating #{CTAGS_FILE}"
    system("mv #{CTAGS_TMP} #{CTAGS_FILE}")
  end

  def ctags?
    system("which ctags > /dev/null")
  end

  def gemfile?
    File.exist?('.gemfile')
  end

  def ctags_paths
    "#{cwd} #{gem_path}"
  end

  def cwd
    "."
  end

  def gem_path
    "#{home}/.rbenv/versions/#{version}/lib/ruby/gems"
  end

  def home
    ENV["HOME"]
  end

  def version
    `rbenv version`.chomp.split(' ').first
  end
end

TagBuilder.new.call
```

Make sure you update permissions to executable!
```bash
chmod +x .git/hooks/post-commit
```

This script compiles ctags in your current checkout and gemfiles installed and managed by rbenv. And yeah, it's triggered after a commit!

Test this in vim with with `ctrl + ]` to "follow a link", and `ctrl + o` to return back to the buffer you came from.

[1.1]: http://help.eclipse.org/neon/topic/org.eclipse.platform.doc.isv/reference/api/org/eclipse/ui/views/navigator/package-summary.html
[2.1]: http://ricostacruz.com/til/navigate-code-with-ctags
[3.1]: https://godoc.org/golang.org/x/tools/cmd/guru
[3.2]: https://github.com/fatih/vim-go
[4.1]: https://sourcegraph.com
[4.2]: https://github.com/lazywei/vim-sourcegraph
[5]: https://github.com/tpope/vim-fugitive
[6]: http://githooks.com/
