---
title: "Encrypted Directories"
date: "2017-06-03T18:24:40+12:00"
summary: A Ruby script to encrypt or decrypt directories with GPG.
tags:
  - script
  - ruby
  - crypto
---

In the *nix ecosystem, there are a lot of tools that are great to
chain together. I'd encourage everyone to learn their tools, but
sometimes the tools are complex, and you only want to do something
simple.

I keep forgetting how to crypto. What I generally want to do is
encrypt and sign a directory, then pull that same directory later.

Here's the script I threw together today. It takes one argument,
either a directory to encrypt, or a gnupg encrypted tar gzipped
directory and extracts it.

```ruby
#!/usr/bin/env ruby

# gpg_folder - Script to quickly tar, compress and encrypt directories

# GpgFolder service drives the OS to perform encrypt/decrypt methods
GpgFolder = Struct.new(:opts) do
  def call
    return false if opts.errors.any?

    if opts.encrypt?
      do_encrypt
    elsif opts.decrypt?
      do_decrypt
    else
      raise ";_; --not sure what I should do now"
    end

    true
  end

  def print_errors
    puts "Computer says no"
    errors.each do |error|
      puts "* #{error}"
    end
  end

  private

  def do_encrypt
    run("tar -cz #{opts.dir_name} | gpg --sign -o #{opts.archive_name}")
  end

  def do_decrypt
    run("gpg -d #{opts.archive_name} | tar xz")
  end

  def run(cmd)
    puts cmd
    system(cmd)
  end
end

# Interprets how the application was called, used to direct GpgFolder service
Options = Struct.new(:dir_or_file) do
  CRYPT_EXTENSION = /\.tgz.gpg\z/

  def errors
    @errors ||= [].tap do
      unless encrypt? ^ decrypt?
        errors << "Please specify a directory to encrypt, or a file ending in .tgz.gpg"
      end
    end
  end

  def encrypt?
    !dir_or_file.nil? && !decrypt? && File.directory?(dir_or_file)
  end

  def decrypt?
    !dir_or_file.nil? && dir_or_file.match(CRYPT_EXTENSION)
  end

  def dir_name
    if encrypt?
      dir_or_file
    else
      dir_or_file.sub(CRYPT_EXTENSION, '')
    end
  end

  def archive_name
    if encrypt?
      "#{dir_or_file}.tgz.gpg"
    else
      dir_or_file
    end
  end
end

opts = Options.new(ARGV.first)
service = GpgFolder.new(opts)
service.call or service.print_errors
```
