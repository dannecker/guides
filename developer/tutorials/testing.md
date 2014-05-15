---
title: Testing Colibri Applications
section: advanced
---

## Overview

The Colibri project currently uses [RSpec](http://rspec.info) for all of its tests. Each of the gems that makes up Colibri has a test suite that can be run to verify the code base.

The Colibri test code is an evolving story. We started out with RSpec, then switched to Shoulda and now we're back to RSpec. RSpec has evolved considerably since we first tried it. When looking to improve the test coverage of Colibri we took another look at RSpec and it was the clear winner in terms of strength of community and documentation.

## Testing Colibri Components

Colibri consists of several different gems (see the [Source Code Guide](navigating#layout-and-structure) for more details.) Each of these gems has its own test suite which can be found in the `spec` directory. Since these gems are also Rails engines, they can't really be tested in complete isolation - they need to be tested within the context of a Rails application.

You can easily build such an application by using the Rake task designed for this purpose, running it inside the component you want to test:

```bash
$ bundle exec rake test_app
```

This will build the appropriate test application inside of your `spec` directory. It will also add the gem under test to your `Gemfile` along with the `colibri_core` gem (since all of the gems depend on this.)

This rake task will regenerate the application (after deleting the existing one) each time you run it. It will also run the migrations for you automatically so that your test database is ready to go. There is no need to run `rake db:migrate` or `rake db:test:prepare` after running `test_app`.

### Running the Specs

Once your test application has been built, you can then run the specs in the standard RSpec manner:

```bash
$ bundle exec rspec spec
```

We also set up a build script that mimics what our build server performs. You can run it from the root of the Colibri project like this:

```bash
$ bash build.sh
```

If you wish to run spec for a single file then you can do so like this:

```bash
$ bundle exec rspec spec/models/colibri/state_spec.rb
```

If you wish to test a particular line number of the spec file then you can do so like this:

```bash
$ bundle exec rspec spec/models/colibri/state_spec.rb:7
```

### Using Factories

Colibri uses [factory_girl](https://github.com/thoughtbot/factory_girl) to create valid records for testing purpose. All of the factories are also packaged in the gem. So if you are writing an extension or if you just want to play with Colibri models then you can use these factories as illustrated below.

```bash
$ rails console
$ require 'colibri/testing_support/factories'
```

The `colibri_core` gem has a good number of factories which can be used for testing. If you are writing an extension or just testing Colibri you can make use of these factories.

## Testing Your Colibri Application

Currently, Colibri does not come with any tests that you can install into your application. What we would advise doing instead is either copying the tests from the components of Colibri and modifying them as you need them.

$$$
Rewrite the preceding paragraph - either copying/modifying or...?
$$$
