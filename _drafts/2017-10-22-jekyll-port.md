---
title: "Messing with Jekyll"
layout: post
---

This is happening mainly because I'm fed up with plone, which is the framework
we currently use at Idiap.  It's not that there's anything fundamentally wrong
with plone, rather that I've got several other ways of adding web content that
are more convenient under markdown / git.

Jekyll is what GitHub recommends; it flies with markdown.  Seems reasonable.

It doesn't sit too easily on a Mac: It's all built around ruby gems; the jekyll
gem is not compatible with the installed version of ruby.  So homebrew installs
a new version of ruby.  Feels like it's gonna be a mess when macos updates.  Ho
hum.

Edit: The process on the Idiap cluster points to perhaps a better way to manage
the gems:

* Install `ruby-dev`
* `gem install jekyll bundler --user-install`
* Set `PATH=$PATH:$HOME/.gem/ruby/2.1.0/bin`
* Run `bundle config --global path $HOME/.gem/ruby/2.1.0`

This will cause bundle to install to the user location thereafter.
