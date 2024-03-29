---
title: "Developer Tips and Tricks"
section: advanced
---

This guide presents accumulated wisdom from person-years of Colibri use.

### Upgrade Considerations

#### The important commands

`colibri -update` was removed in favor of Bundler.

Before updating, you will want to ensure the installed colibri gem is
up-to-date by modifying `Gemfile` to match the new colibri version and
run `bundle update`.

Thanks to Rails 3.1 Mountable Engine, the update process is
"non-destructive" than in previous versions of Colibri. The core files are encapsulated
separately from sandbox, thus upgrading to newer files will not override nor replace
sandbox's customized files.

This makes it easier to see when and how some file has changed – which
is often useful if you need to update a customized version.

#### Dos and Don'ts

!!!
Try to avoid modifying `config/boot.rb` and
`config/environment.rb`: use [initializers](#initializers) instead.
!!!

#### Tracking changes for overridden code

Be aware that core changes might have an impact on the components you
have overridden in your project.
You might need to patch your local copies, or ensure that such copies
interact correctly with changed code (e.g. using appropriate ids in HTML to allow the JavaScript to
work).

If you can help us generalise the core code so that your preferred
effect is achieved by altering a few parameters, this will be more useful than duplicating several
files. Ideas and suggestions are always welcome.

#### Initializers

Initializers are run during startup, and are the recommended way to
execute certain settings. You can put initializers in extensions, thus have a way to execute
extension-specific configurations.

See the [extensions guide](extensions_tutorial.html#extension-initializers) for
more information.

### Debugging techniques

#### Use tests!

Use `rake spec` and `rake test` to test basic functioning after you've
made changes.

#### Analysing crashes on a non-local machine

If you're testing on a server, whether in production or development
mode, the following code in one
of your `FOO_extension.rb` files might save some time. It triggers
local behaviour for users who have
an admin role. One useful consequence is that uncaught exceptions will
show the detailed error page
instead of `404.html`, so you don't have to hunt through the server
logs.

<% ruby do %>
    Colibri::BaseController.class_eval do
      def local_request?
        ENV["RAILS_ENV"] !="production" || current_user.present? &&
          current_user.has_role?(:admin)
      end
    end
<% end %>

### Managing large projects

#### To fork or not to fork…

Suppose there's a few details of Colibri that you want to override due to
personal or client preference,
but which aren't the usual things that you'd override (like views) - so
something like tweaks to the models or controllers.

You could hide these away in your site extension, but they could get
mixed up with your real site customizations. You could also fork Colibri and run your site on this
forked version, but this can also be a headache to get right. There's also the hassle of tracking
changes to `colibri/master` and pulling them into your project at the right time.

So here's a compromise: have an extra extension, say `colibri-tweaks`, to
contain your small collection of modified files, which is loaded first in the extension order. The
benefits are:

-   it's clear what you are overriding, and easier to check against core
    changes
-   you can base your project on an official gem release or a
    `colibri/master` commit stage
-   such tweaks can become part of your client site project and be
    managed with SCM etc.

If you find yourself wanting extensive changes to core, this technique
probably won't work so well.
But then again, if this is the case, then you probably want to look
seriously at splitting some
code off into stand-alone extensions and then see whether any of the
other code should be contributed to the core.
