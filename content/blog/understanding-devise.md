---
layout: post
title: "Understanding Devise"
date: 2013-04-07T00:40:00
comments: true
categories: rails devise
---

Ever wonder how Devise authentication works end-to-end? If you've ever needed to work with its internals or extend its functionality, having a full understanding of how it handles authentication is essential.

### The Request

Of course, the entire process begins with an HTTP request, which makes its way to your HTTP server. From there, the request needs to be passed to your application.

### Rack Middleware

Rack acts as the interface between your HTTP server and Rails application. This isn't just a simple exchange; Rack provides hooks for _middleware_, classes that can read and modify requests before they hit your application. Rails adds several middleware classes, one of which is essential to authentication: `ActionDispatch::Session::CookieStore`. This class creates a session (or loads one from the client's cookie) and adds it to the request so that it's readily available to anything downstream.

### Warden

Since the session is now available, the first layers of authentication can begin. Warden is a low-level authentication library for Rack, and sits in the stack after the session middleware. At this point, it only adds a "warden" object to the request and passes it downstream. Since the decision to run authentication can depend on which controller or view is being requested, Warden doesn't need to do any additional work, letting the controller invoke authentication later on, if needed.

You might wonder why Warden belongs in the middleware stack if it's not used until the request is being processed by your controllers. Rails could just as easily load a Warden class that works at the application layer. This is done to decouple the authentication layer and your application, which can greatly simplify single sign-on and authentication sharing with other services using Warden.

### Rails Application

At this point, your application is happily handling the request, which will have made its way to a controller. If the user is trying to access a controller or view that requires authentication, you'll have added a call to `authenticate_user!` as a `before_filter`. This helper is dynamically generated by the following devise method:

```ruby
def self.define_helpers(mapping) #:nodoc:
  mapping = mapping.name

  class_eval <<-METHODS, __FILE__, __LINE__ + 1
    def authenticate_#{mapping}!(opts={})
      opts[:scope] = :#{mapping}
      warden.authenticate!(opts) if !devise_controller? || opts.delete(:force)
    end

    def #{mapping}_signed_in?
      !!current_#{mapping}
    end

    def current_#{mapping}
      @current_#{mapping} ||= warden.authenticate(:scope => :#{mapping})
    end

    def #{mapping}_session
      current_#{mapping} && warden.session(:#{mapping})
    end
  METHODS

  ActiveSupport.on_load(:action_controller) do
    helper_method "current_#{mapping}", "#{mapping}_signed_in?", "#{mapping}_session"
  end
end
```

The helper method is created using the model name. Since most applications implement Devise with a `User` model, the helper is often defined as `authenticate_user!`. This bit of code also uses the class name as the Warden authentication _scope_. Warden scopes can be thought of as authentication namespaces; they allow you to run multiple distinct authentications in a single session. Devise uses a scope for each authentication class. Going forward, we'll assume we have two authentication classes as follows:

```ruby
class User < ActiveRecord::Base
  devise :database_authenticatable
end

class Admin < ActiveRecord::Base
  devise :ldap_authenticatable
end
```

### Strategies

Warden introduces the concept of _strategies_, which are methods of authentication. The `User` and `Admin` classes use database and directory strategies, respectively. Their implementations are either shipped as part of Devise itself or as 3rd party authentication libraries. Strategies need to be loaded before they can be used; this is accomplished with a call to `Warden::Strategies.add`, typically done in the strategy file itself after the class declaration.

### Warden Authentication

Once `authenticate_user!` is called, the warden object is asked to authenticate the request (as seen in the call to `warden.authenticate!(opts)`). The `warden` keyword is just a Devise method that maps to the warden object previously added to the request.

Before it bothers doing any work, Warden checks to see if the request has already been authenticated for the specified scope (which is passed in the `opts` hash). If so, it simply returns the authenticated user object extracted from the session. If not, Warden will authenticate the request trying each of the strategies configured for the given scope. Devise adds these as defaults during its initialization:

```ruby
def self.configure_warden!
  @@warden_configured ||= begin
    warden_config.failure_app   = Devise::Delegator.new
    warden_config.default_scope = Devise.default_scope
    warden_config.intercept_401 = false

    Devise.mappings.each_value do |mapping|
      warden_config.scope_defaults mapping.name, :strategies => mapping.strategies
    end

    @@warden_config_block.try :call, Devise.warden_config
    true
  end
end
```

That the strategies are set as defaults is the reason the aforementioned call to `warden.authenticate!` doesn't require an explicit strategy. Warden looks up the default strategies for the given scope when none are specified.

The strategies are then run until one of three things happen:

* A strategy succeeds and returns a user object.
* All strategies are attempted, but none are successful.
* A strategy explicitly fails, preventing any remaining strategies from being run.

If the authentication fails, Warden calls `throw`, which invokes a _failure application_. Conveniently enough, Devise sets one up during its initialization that redirects to the sign in page when called. If the authentication succeeds, the return value of the `authenticate!` method bubbles up the stack and is returned by `authenticate_user!`.

### Conclusion

This is just a brief overview of Devise's authentication design; there is a _lot_ more going on in both the Devise and Warden libraries. That being said, an understanding of these fundamental concepts should serve you well should you need to debug or modify the behaviour of Devise.

Happy coding!