---
title: Upgrading Colibri from 1.1.x to 1.2.x
section: upgrades
---

This guide covers upgrading a 1.1.x Colibri store, to a 1.2.x store. This
guide has been written from the perspective of a blank Colibri 1.1.x store with
no extensions.

If you have extensions that your store depends on, you will need to manually
verify that each of those extensions work within your 1.2.x store once this
upgrade is complete. Typically, extensions that are compatible with this
version of Colibri will have a 1-2-stable branch.

## Upgrade Colibri

For best results, use the 1-2-stable branch from GitHub:

```ruby
gem 'colibri', :github => 'colibri/colibri', :branch => '1-2-stable'```

Run `bundle update colibri`. 

## Authentication dependency

In this release, the `colibri_auth` component was moved out of the main set of
gems into an extension, called `colibri_auth_devise`. If you want to continue using Colibri's authentication, then you will need to specify this extension as a dependency in your `Gemfile`:

```ruby
gem 'colibri_auth_devise', :github => 'colibri/colibri_auth_devise', :branch => '1-2-stable'```

Run `bundle install` to install this extension.

### Rename current_user to current_colibri_user

To ensure that Colibri does not conflict with any authentication provided by the application, Colibri has renamed its `current_user` variable to `current_colibri_user`. You should make this change wherever necessary within your application.

Similar to this, any references to `@user` are now `@colibri_user`.

## Copy and run migrations

Copy over the migrations from Colibri (and any other engine) and run them using
these commands:

    rake railties:install:migrations
    rake db:migrate

This may copy over additional migrations from colibri_auth_devise and run them as well.

## Read the release notes

For information about changes contained with this release, please read the [1.2.0 Release Notes](http://guides.usoft.com.ua/colibri/release_notes/colibri_1_2_0.html).

## Verify that everything is OK

Click around in your store and make sure it's performing as normal. Fix any deprecation warnings you see.
