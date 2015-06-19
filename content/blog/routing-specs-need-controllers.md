---
layout: post
title: "Routing Specs Need Controllers"
date: 2012-11-28T22:43:00
comments: true
categories: rspec rails routing
---
Once you get used to the TDD/BDD style of development, you learn to write small, focused tests and then implement the corresponding functionality. This usually works well, but you might run into an issue when writing routing tests. If write a test for a route pointing to a controller that doesn't exist, you'll get a "No route matches _/your/route/here_" error, which is rather cryptic considering the route displays when running `rake routes`. Create the corresponding controller to fix this issue. It's a shame that the routing tests depend on the corresponding controllers, but the methods used in their tests (get/put/post/delete) rely on their existence.
