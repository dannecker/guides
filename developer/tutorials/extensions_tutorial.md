---
title: Extensions
section: tutorial
---

## Introduction

This tutorial continues where we left off in the [Getting Started](getting_started_tutorial) tutorial. Now that we have a basic Colibri store up and running, let's spend some time customizing it. The easiest way to do this is by using Colibri extensions.

### What is a Colibri Extension?

Extensions are the primary mechanism for customizing a Colibri site. They provide a convenient mechanism for Colibri developers to share reusable code with one another. Even if you do not plan on sharing your extensions with the community, they can still be a useful way to reuse code within your organization. Extensions are also a convenient mechanism for organizing and isolating discrete chunks of functionality.

### Finding Useful Colibri Extensions in the Extension Registry

The [Colibri Extension Registry](http://usoft.com.ua/colibri/extensions) is a searchable collection of Colibri Extensions written and maintained by members of the [Colibri Community](http://usoft.com.ua/colibri/community). If you need to extend your Colibri application's functionality, be sure to have a look in the Extension Registry first; you may find an extension that either implements what you need or provides a good starting point for your own implementation. If you write an extension and it might be useful to others, [publish it in the registry](http://usoft.com.ua/colibri/extensions/new) and people will be able to find it and contribute as well.

## Installing an Extension

We are going to be adding the [colibri_fancy](https://github.com/colibri/colibri_fancy) extension to our store. ColibriFancy is a theme so it only changes the look and feel of the application. Extensions can also add models, controllers, and views to create new functionality, but colibri_fancy is intended as a starting point to show how a barebones Colibri application can be easily modified to give a nice look and feel. As a special bonus it's fully responsive and looks good on mobile devices as well as on larger screens.

There are three steps we need to take to install colibri_fancy.

First, we need to add the gem to the bottom of our `Gemfile`:

```ruby
gem 'colibri_fancy', :git => 'git://github.com/colibri/colibri_fancy.git', :branch => '2-1-stable'
```
****

Note that if you are using the edge version of colibri, you should omit the branch parameter to get the latest version of colibri_fancy. Alternatively, you should select the version of colibri_fancy that corresponds with your version of colibri.

***
If you are using a 2.1.x version of Colibri, the above line will work fine. If you're using a 2.0.x version of Colibri, you'll need to change the "branch" option to point to the "2-0-stable" branch. If you're using the "master" branch of Colibri, change the "branch" argument for "colibri_fancy" to be "master" as well.
***

Now, let's install the gem via Bundler with the following command:

```bash
$ bundle install
```

Finally, let's copy over the required migrations and assets from the extension with the following command:

```bash
$ bundle exec rails g colibri_fancy:install
```

Answer **yes** when prompted to run migrations.

When the last command is done running, you can start your application again and navigate to `http://localhost:3000` to see our brand new theme.

## Creating an Extension

### Getting Started

Let's build a simple extension. Suppose we want the ability to mark certain products as being on sale. We'd like to be able to set a sale price on a product and show products that are on sale on a separate products page. This is a great example of how an extension can be used to build on the solid Colibri foundation.

So let's start by generating the extension. Run the following command from a directory of your choice outside of our Colibri application:

```bash
$ colibri extension simple_sales
```

This creates a `colibri_simple_sales` directory with several additional files and directories. After generating the extension make sure you change to its directory:

```bash
$ cd colibri_simple_sales
```

### Adding a Sale Price to Variants

The first thing we need to do is create a migration that adds a sale_price column to [variants](http://guides.usoft.com.ua/colibri/products_and_variants.html#what-is-a-variant).

We can do this with the following command:

```bash
rails g migration add_sale_price_to_colibri_variants sale_price:decimal
```

$$$
Make above generator actually work in extension directories
$$$

Because we are dealing with prices, we need to now edit the generated migration to ensure the correct precision and scale. Edit the file `db/migrate/XXXXXXXXXXX_add_sale_price_to_colibri_variants.rb` so that it contains the following:

```ruby
class AddSalePriceToColibriVariants < ActiveRecord::Migration
  def change
    add_column :colibri_variants, :sale_price, :decimal, :precision => 8, :scale => 2
  end
end
```

### Adding Our Extension to the Colibri Application

Before we continue development of our extension, let's add it to the Colibri application we created in the [last tutorial](/developer/getting_started.html). This will allow us to see how the extension works with an actual Colibri store while we develop it.

Within the `mystore` application directory, add the following line to the bottom of our `Gemfile`:

```ruby
gem 'colibri_simple_sales', :path => '../colibri_simple_sales'
```

You may have to adjust the path somewhat depending on where you created the extension. You want this to be the path relative to the location of the `mystore` application.

Once you have added the gem, it's time to bundle:

```bash
$ bundle install
```

Finally, let's run the `colibri_simple_sales` install generator to copy over the migration we just created (answer **yes** if prompted to run migrations):

```bash
$ rails g colibri_simple_sales:install
```

### Adding a Controller Action to HomeController

Now we need to extend `Colibri::HomeController` and add an action that selects "on sale" products.

Make sure you are in the `colibri_simple_sales` root directory and run the following command to create the directory structure for our controller decorator:

```bash
$ mkdir -p app/controllers/colibri
```

Next, create a new file in the directory we just created called `home_controller_decorator.rb` and add the following content to it:

```ruby
module Colibri
  HomeController.class_eval do
    def sale
      @products = Product.joins(:variants_including_master).where('colibri_variants.sale_price is not null').uniq
    end
  end
end
```

This will select just the products that have a variant with a `sale_price` set.

We also need to add a route to this action in our `config/routes.rb` file. Let's do this now. Update the routes file to contain the following:

```ruby
Colibri::Core::Engine.routes.draw do
  get "/sale" => "home#sale"
end
```

### Viewing On Sale Products

#### Setting the Sale Price for a Variant

Now that our variants have the attribute `sale_price` available to them, let's update the sample data so we have at least one product that is on sale in our application. We will need to do this in the rails console for the time being, as we have no admin interface to set sale prices for variants. We will be adding this functionality in the [next tutorial]() in this series, Deface overrides.

So, in order to do this, first open up the rails console:

```bash
$ rails console
```

Now, follow the steps I take in selecting a product and updating its master variant to have a sale price. Note, you may not be editing the exact same product as I am, but this is not important. We just need one "on sale" product to display on the sales page.

```irb
> product = Colibri::Product.first
=> #<Colibri::Product id: 107377505, name: "Colibri Bag", description: "Lorem ipsum dolor sit amet, consectetuer adipiscing...", available_on: "2013-02-13 18:30:16", deleted_at: nil, permalink: "colibri-bag", meta_description: nil, meta_keywords: nil, tax_category_id: 25484906, shipping_category_id: nil, count_on_hand: 10, created_at: "2013-02-13 18:30:16", updated_at: "2013-02-13 18:30:16", on_demand: false>

> variant = product.master
=> #<Colibri::Variant id: 833839126, sku: "SPR-00012", weight: nil, height: nil, width: nil, depth: nil, deleted_at: nil, is_master: true, product_id: 107377505, count_on_hand: 10, cost_price: #<BigDecimal:7f8dda5eebf0,'0.21E2',9(36)>, position: nil, lock_version: 0, on_demand: false, cost_currency: nil, sale_price: nil>

> variant.sale_price = 8.00
=> 8.0

> variant.save
=> true
```

### Creating a View

Now we have at least one product in our database that is on sale. Let's create a view to display these products.

First, create the required views directory with the following command:

```bash
$ mkdir -p app/views/colibri/home
```

Next, create the file `app/views/colibri/home/sale.html.erb` and add the following content to it:

```erb
<div data-hook="homepage_products">
  <%%= render 'colibri/shared/products', :products => @products %>
</div>
```

If you navigate to `http://localhost:3000/sale` you should now see the product(s) listed that we set a `sale_price` on earlier in the tutorial. However, if you look at the price, you'll notice that it's not actually displaying the correct price. This is easy enough to fix and we will cover that in the next section.

### Decorating Variants

Let's fix our extension so that it uses the `sale_price` when it is present.

First, create the required directory structure for our new decorator:

```bash
$ mkdir -p app/models/colibri
```

Next, create the file `app/models/colibri/variant_decorator.rb` and add the following content to it:

```ruby
module Colibri
  Variant.class_eval do
    alias_method :orig_price_in, :price_in
    def price_in(currency)
      return orig_price_in(currency) unless sale_price.present?
      Colibri::Price.new(:variant_id => self.id, :amount => self.sale_price, :currency => currency)
    end
  end
end
```

Here we alias the original method `price_in` to `orig_price_in` and override it. If there is a `sale_price` present on the product's master variant, we return that price. Otherwise, we call the original implementation of `price_in`.

### Testing Our Decorator

It's always a good idea to test your code. We should be extra careful to write tests for our Variant decorator since we are modifying core Colibri functionality. Let's write a couple of simple unit tests for `variant_decorator.rb`

#### Generating the Test App

An extension is not a full Rails application, so we need something to test our extension against. By running the Colibri `test_app` rake task, we can generate a barebones Colibri application within our `spec` directory to run our tests against.

We can do this with the following command from the root directory of our extension:

```bash
$ bundle exec rake test_app
```

After this command completes, you should be able to run `rspec` and see the following output:

```bash
No examples found.

Finished in 0.00005 seconds
0 examples, 0 failures
```

Great! We're ready to start adding some tests. Let's replicate the extension's directory structure in our spec directory by running the following command

```bash
$ mkdir -p spec/models/colibri
```

Now, let's create a new file in this directory called `variant_decorator_spec.rb` and add the following tests to it:

```ruby
require 'spec_helper'

describe Colibri::Variant do
  describe "#price_in" do
    it "returns the sale price if it is present" do
      variant = create(:variant, :sale_price => 8.00)
      expected = Colibri::Price.new(:variant_id => variant.id, :currency => "USD", :amount => variant.sale_price)

      result = variant.price_in("USD")

      result.variant_id.should == expected.variant_id
      result.amount.to_f.should == expected.amount.to_f
      result.currency.should == expected.currency
    end

    it "returns the normal price if it is not on sale" do
      variant = create(:variant, :price => 15.00)
      expected = Colibri::Price.new(:variant_id => variant.id, :currency => "USD", :amount => variant.price)

      result = variant.price_in("USD")

      result.variant_id.should == expected.variant_id
      result.amount.to_f.should == expected.amount.to_f
      result.currency.should == expected.currency
    end
  end
end
```

These specs test that the `price_in` method we overrode in our `VariantDecorator` returns the correct price both when the sale price is present and when it is not.

## Versioning your extension

Different versions of Colibri may act differently with your extension. It's advisable to keep different branches of your extension actively maintained for the different branches of Colibri so that your extension will work with those different versions.

It's advisable that your extension follows the same versioning pattern as Colibri itself. If your extension is compatible with Colibri 2.0.x, then create a `2-0-stable` branch on your extension and advise people to use that branch for your extension. If it's only compatible with 1.3.x, then create a 1-3-stable branch and advise the use of that branch.

Having a consistent branching naming scheme across Colibri and its extensions will reduce confusion in the long run.

## Summary

In this tutorial you learned how to both install extensions and create your own. A lot of core Colibri development concepts were covered and you gained exposure to some of the Colibri internals.

In the [next part](deface_overrides_tutorial) of this tutorial series, we will cover [Deface](https://github.com/colibri/deface) overrides and look at ways to improve our current extension.
