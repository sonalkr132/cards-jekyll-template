---
layout: post
title: "Plugin system"
image: '/assets/img/'
main-class: 'bundler'
tags:
- ruby
- bundler
introduction: 'Historically, Bundler has only allowed new features to be implemented as part of the core Bundler project. After a few weeks of planning..'
---

#### Plugin system

Historically, Bundler has only allowed new features to be implemented as part of the core Bundler project. After a few weeks of planning work, we're ready to start work on command plugins (like `bundle grep`), on source plugins (like `gem "rack", svn: "http://some-url.com"`), and on lifecycle hooks (pre-install, post-install, and others).

Declaring a plugin in the Gemfile should look something like this:

```
source "https://rubygems.org"

# Hook and command plugin names map to gem names:
plugin "foo" # this plugin is in gem "bundler-foo"

# Plugins have the same options that gems have, like :git and :source
plugin "bar", git: "https://github.com/bundler/bundler-bar.git"

# Source plugins will have to be installed and loaded in advance for us to be able to process the rest of the Gemfile. Let's declare up front that those plugin gems can't have any dependencies of their own, and that they have a different API, maybe something like this:

# this type option maps to the plugin named "bundler-source-svn"
source "svn://svnhub.com/name/repo", type: "svn" do
  gem "my-cool-gem"
end
```

Lifecycle hooks will be made available to Bundler plugins that are either installed by a user or listed in the Gemfile using the `plugin` syntax.

Bundler will need to require a specific file in those gems, probably `bundler-plugin.rb`, and allow plugins to use that file to register for lifecycle hooks. A sample `bundler-plugin.rb` file might look like this:

```ruby
Bundler.add_hook("after-install") do |gem|
  require 'plugin_stuff'
  PluginStuff.new.process_gem(gem)
end
```

As of 1.9, we have subcommands that are simply on the path, like a gem that provides an executable named `bundle-foo` will be invoked if you run `bundle foo bar`. This is for commands that we want to automatically download and install when you install a Gemfile.

I think the simplest possible implementation of this idea is JUST to install the gem. That would mean this card depends only on the ability to declare plugins in the Gemfile card: https://trello.com/c/xaMdMwmq


Source plugins are hard, because they have to be loaded before the rest of the gemfile can even be evaluated. To begin with, let's require that source plugins must have no dependencies, and must be declared a different way:

```
# this type option maps to the plugin named "bundler-source-svn"
source "svn://svnhub.com/name/repo", type: "svn" do
  gem "my-cool-gem"
end
```

We can evaluate the Gemfile once with only the `source` method defined, looking for `type` arguments so that we know what source plugins need to be downloaded, installed, and loaded before we can evaluate the entire Gemfile. We will still need to handle cases like `--local` when the source plugin isn't installed.

Source plugins will need to register themselves with Bundler, but we're not yet sure where the hooks are. Extracting the Path and Git code into source plugins that are simply included with Bundler would be a good start.

A few years ago, I wrote [a blog post with ideas for plugins](http://andre.arko.net/2012/07/23/towards-a-bundler-plugin-system/). There have been several tickets discussing plugins: [one](https://github.com/bundler/bundler/issues/1945) [two](https://github.com/bundler/bundler-features/issues/8) [three](https://github.com/bundler/bundler/issues/3463). There was also a GSoC project last year that produced [a proof-of-concept pull request](https://github.com/bundler/bundler/pull/3807), but the project did not reach completion.

* **Prerequisites**: Familiarity with Ruby and Bundler
* **Programming areas include**: Ruby (competence or willingness to learn)
* **Estimated difficulty level**: medium to hard
* **Potential mentors**: @indirect, @andremedeiros
