---
title: Upgrading Colibri from 2.0.x to 2.1.x
section: upgrades
---

This guide covers upgrading a 2.0.x Colibri store, to a 2.1.x store. This
guide has been written from the perspective of a blank Colibri 2.0.x store with
no extensions.

If you have extensions that your store depends on, you will need to manually
verify that each of those extensions work within your 2.1.x store once this
upgrade is complete. Typically, extensions that are compatible with this
version of Colibri will have a 2-1-stable branch.

This is the first Colibri release which supports Rails 4 exclusively. Colibri
releases after this point will continue to support Rails 4 only.

## Upgrade Rails

For this Colibri release, you will need to upgrade your Rails version to at least 4.0.0.

It is recommended to read through the [Upgrading Ruby on Rails
guide](http://guides.rubyonrails.org/upgrading_ruby_on_rails.html#upgrading-
from-rails-3-2-to-rails-4-0) to learn what needs to be done for your
application to migrate to Rails 4.

```ruby
gem 'rails', '~> 4.0.0'```

## Upgrade Colibri

For best results, use the 2-1-stable branch from GitHub:

```ruby
gem 'colibri', :github => 'colibri/colibri', :branch => '2-1-stable'```

Run `bundle update colibri`.

## Copy and run migrations

Copy over the migrations from Colibri (and any other engine) and run them using
these commands:

    rake railties:install:migrations
    rake db:migrate

## Read the release notes

For information about changes contained with this release, please read the [2.1.0 Release Notes](http://guides.usoft.com.ua/colibri/release_notes/colibri_2_1_0.html).

## Verify that everything is OK

Click around in your store and make sure it's performing as normal. Fix any deprecation warnings you see.
