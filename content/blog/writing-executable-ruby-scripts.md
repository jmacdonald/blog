---
layout: post
title: "Writing Executable Ruby Scripts"
date: 2012-04-14T15:35:00
comments: true
categories: ruby
---

Ruby is awesome. That being said, the typical way ruby scripts are executed _sucks_:

``` bash
ruby my_script.rb
```

If you're planning on running a script more than once, that syntax is pretty abhorrent. We really ought to be using a _shebang_ line to tell the shell that this script should be run by a particular interpreter:

``` bash
#!/usr/bin/ruby

# Write awesome Ruby code here.
```

This is a lot better than our original approach, but it's definitely not perfect. I'm using rbenv, and the aforementioned syntax won't work with my setup. Even if I did point the shebang line to my ruby executable, it's then tied to a specific version of Ruby, which may not be available at a later point in time.

The _right_ way to do this is to use the ``env`` command, which when passed an argument, will try to execute that argument using the current user's environment. In our case, running the ``ruby`` executable in this context will take advantage of rbenv or RVM if installed, but will still work in their absence.

``` bash
#!/usr/bin/env ruby
```

This is the most portable and effective way to run ruby scripts with a first-class-citizen syntax. Drop the ``.rb`` from the file name, throw it in a folder in your ``$PATH`` and you can execute the script from anywhere.
