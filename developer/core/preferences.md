---
title: "Preferences"
section: core
---

## Overview

Colibri Preferences support general application configuration and preferences per model instance. Colibri comes with preferences for your store like `site_name` and`site_url`. Additional preferences can be added by your application or included extensions.

Preferences for models can be added without modifying the database. All instances will use the default value unless a value has been set for a specific record. For example, you could add a preference to `User` for e-mail notifications. Users would have the ability to modify this value without adding a column to your database table.

Extensions may add to the Colibri General Settings or create their own namespaced preferences.

The first several sections of this guide describe preferences in a very general way. If you're just interested in making modifications to the existing preferences, you can skip ahead to the [Configuring Colibri Preferences section](#configuring-colibri-preferences). If you would like a more in-depth understanding of the underlying concepts used by the preference system, please read on.

### Motivation

Preferences for models within an application are very common. Although the rule of thumb is to keep the number of preferences available to a minimum, sometimes it's necessary if you want users to have optional preferences like disabling e-mail notifications.

General application settings like `site_name` are also available to help customize your Colibri store.

Both use cases are handled by Colibri Preferences. They are easy to define, provide quick cached reads, persist across restarts and do not require additional columns to be added to your models' tables.

## General Settings

Colibri comes with many application-wide preferences. They are defined in `core/app/models/colibri/app_configuration.rb` and made available to your code through `Colibri::Config`, e.g., `Colibri::Config.site_name`.

A limited set of the general settings are available in the admin interface of your store (`/admin/general_settings`).

You can add additional preferences under the `colibri/app_configuration` namespace or create your own subclass of `Preferences::Configuration`.

```ruby
# These will be saved with key: colibri/app_configuration/hot_salsa
Colibri::AppConfiguration.class_eval do
  preference :hot_salsa, :boolean
  preference :dark_chocolate, :boolean, :default => true
  preference :color, :string
  preference :favorite_number
  preference :language, :string, :default => 'English'
end

# Colibri::Config is an instance of Colibri::AppConfiguration
Colibri::Config.hot_salsa = false

# Create your own class
# These will be saved with key: kona/store_configuration/hot_coffee
Kona::StoreConfiguration < Preferences::Configuration
  preference :hot_coffee, :boolean
  preference :color, :string, :default => 'black'
end

KONA::STORE_CONFIG = Kona::StoreConfiguration.new
puts KONA::STORE_CONFIG.hot_coffee
```

## Defining Preferences

You can define preferences for a model within the model itself:

```ruby
class User < ActiveRecord::Base
  preference :hot_salsa, :boolean
  preference :dark_chocolate, :boolean, :default => true
  preference :color, :string
  preference :favorite_number, :integer
  preference :language, :string, :default => "English"
end
```

In the above model, five preferences have been defined:

* `hot_salsa`
* `dark_chocolate`
* `color`
* `favorite_number`
* `language`

For each preference, a data type is provided. The types available are:

* `boolean`
* `string`
* `password`
* `integer`
* `text`

An optional default value may be defined.

## Accessing Preferences

Once preferences have been defined for a model, they can be accessed either using the shortcut methods that are generated for each preference or the generic methods that are not specific to a particular preference.

### Shortcut Methods

There are several shortcut methods that are generated. They are shown below.

Query methods:

```ruby
user.prefers_hot_salsa? # => false
user.prefers_dark_chocolate? # => false
```

Reader methods:

```ruby
user.preferred_color      # => nil
user.preferred_language   # => "English"
```

Writer methods:

```ruby
user.prefers_hot_salsa = false         # => false
user.preferred_language = "English"    # => "English"
```

Check if a preference is available:

```ruby
user.has_preference? :hot_salsa
```

### Generic Methods

Each shortcut method is essentially a wrapper for the various generic methods shown below:

Query method:

```ruby
user.prefers?(:hot_salsa)       # => false
user.prefers?(:dark_chocolate)  # => false
```

Reader methods:

```ruby
user.preferred(:color)      # => nil
user.preferred(:language)   # => "English"
```

```ruby
user.get_preference :color
user.get_preference :language
```

Writer method:

```ruby
user.set_preference(:hot_salsa, false)     # => false
user.set_preference(:language, "English")  # => "English"
```

### Accessing All Preferences

You can get a hash of all stored preferences by accessing the `preferences` helper:

```ruby
user.preferences # => {"language"=>"English", "color"=>nil}
```

This hash will contain the value for every preference that has been defined for the model instance, whether the value is the default or one that has been previously stored.

### Default and Type

You can access the default value for a preference:

```ruby
user.preferred_color_default # => 'blue'
```

Types are used to generate forms or display the preference. You can also get the type defined for a preference:

```ruby
user.preferred_color_type # => :string
```

## Configuring Colibri Preferences

Up until now we've been discussing the general preference system that was adapted to Colibri. This has given you a general idea of what types of preference features are theoretically supported. Now, let's start to look specifically at how Colibri is using these preferences for configuration.

### Reading the Current Preferences

At the heart of Colibri preferences lies the `Colibri::Config` constant. This object provides general access to the configuration settings anywhere in the application.

These settings can be accessed from initializers, models, controllers, views, etc.

The `Colibri::Config` constant returns an instance of `Colibri::AppConfiguration` which is where the default values for all of the general Colibri preferences are defined.

You can access these preferences directly in code. To see this in action, just fire up `rails console` and try the following:

```ruby
>> Colibri::Config.site_name
=> "Colibri Demo Site"
>> Colibri::Config.admin_products_per_page
=> 10
```

The above examples show the default configuration values for these preferences. The defaults themselves are coded within the `Colibri::AppConfiguration` class.

```ruby
class Colibri::AppConfiguration < Configuration
  #... snip ...
  preference :site_name, :string, :default => 'Colibri Demo Site'
  #... snip ...
end
```

If you are using the default preferences without any modifications, then nothing will be stored in the database. If you set a value for the preference it will save it to `colibri_preferences`. It will use a memory cached version to maintain performance.

### Overriding the Default Preferences

The default Colibri preferences in `Colibri::AppConfiguration` can be changed using the `set` method of the `Colibri::Config` module. For example to set the number of products shown on the products listing in the admin interface we could do the following:

```ruby
>> Colibri::Config.admin_products_per_page = 20
=> 20
>> Colibri::Config.admin_products_per_page
=> 20
```

Here we are changing a preference to something other than the default as specified in `Colibri::AppConfiguration`. In this case the preference system will persist the new value in the `colibri_preferences` table.

### Configuration Through the Colibri Initializer

During the Colibri installation process, an initializer file is created within your application's source code. The initializer is found under `config/initializers/colibri.rb`:

```ruby
Colibri.config do |config|
  # Example:
  # Uncomment to override the default site name.
  # config.site_name = "Colibri Demo Site"
end
```

The `Colibri.config` block acts as a shortcut to setting `Colibri::Config` multiple times. If you have multiple default preferences you would like to override within your code you may override them here. Using the initializer for setting the defaults is a nice shortcut, and helps keep your preferences organized in a standard location.

For example if you would like to change the site name and default locale you can accomplish this by doing the following:

```ruby
Colibri.config do |config|
  config.site_name = 'My Awesome Colibri Site'
end
```

***
Initializing preferences in `config/initializer.rb` will overwrite any changes that were made through the admin user interface when you restart.
***

### Configuration Through the Admin Interface

The Colibri admin interface has several different screens where various settings can be configured. For instance, the `admin/general_settings` URL in your Colibri application can be used to configure the values for the site name and the site URL. This is basically equivalent to calling `Colibri::Config.set(site_name => "Whatever", :site_url => "http://whatever.com")` directly in your Ruby code.

## Site-Wide Preferences

You can define preferences that are site-wide and don't apply to a specific instance of a model by creating a configuration file that inherits from `Colibri::Preferences::Configuration`.

```ruby
class Colibri::MyApplicationConfiguration < Colibri::Preferences::Configuration
  preference :theme, :string, :default => "Default"
  preference :show_splash_page, :boolean
  preference :number_of_articles, :integer
end
```

In the above configuration file, three preferences have been defined:

* theme
* show_splash_page
* number_of_articles

It is recommended to create the configuration file in the `lib/` directory.

***
Extensions can also define site-wide preferences. For more information on using preferences like this with extensions, check out the [Extensions Tutorial](extensions_tutorial).
***

### Configuring Site-Wide Preferences

The recommended way to configure site-wide preferences is through an initializer. Let's take a look at configuring the preferences defined in the previous configuration example.

```ruby
module Colibri
  MyApp::Config = Colibri::MyApplicationConfiguration.new
end

MyApp::Config[:theme] = "blue_theme"
MyApp::Config[:show_spash_page] = true
MyApp::Config[:number_of_articles] = 5```

The `MyApp` name used here is an example and should be replaced with your actual application's name, found in `config/application.rb`.

The above example will configure the preferences we defined earlier. Take note of the second line. In order to set and get preferences using `MyApp::Config`, we must first instantiate the configuration object.

## Colibri Configuration Options

This section lists all of the configuration options for the current version of Colibri.

`address_requires_state`

Will determine if the state field should appear on the checkout page. Defaults to `true`.

`admin_interface_logo`

The path to the logo to display on the admin interface. Can be different from `Colibri::Config[:logo]`. Defaults to `logo/colibri_50.png`.

`admin_products_per_page`

How many products to display on the products listing in the admin interface. Defaults to 10.

`allow_backorder_shipping`

Determines if an `InventoryUnit` can ship or not. Defaults to `false`.

`allow_checkout_on_gateway_error`

Continues the checkout process even if the payment gateway error failed. Defaults to `false`.

`allow_ssl_in_development_and_test`

Enables SSL support in development and test environments. Defaults to `false`.

`allow_ssl_in_production`

Enables SSL support in production environment. Defaults to `true`.

`allow_ssl_in_staging`

Enables SSL support in production environment. Defaults to `true`.

`alternative_billing_phone`

Determines if an alternative phone number should be present for the billing address on the checkout page. Defaults to `false`.

`alternative_shipping_phone`

Determines if an alternative phone number should be present for the shipping address on the checkout page. Defaults to `false`.

`always_put_site_name_in_title`

Determines if the site name (`Colibri::Config[:site_name]`) should be placed into the title. Defaults to `true`.

`attachment_default_url`

Tells `Paperclip` the form of the URL to use for attachments which are missing.

`attachment_path`

Tells `Paperclip` the path at which to store images.

`attachment_styles`

A JSON hash of different styles that are supported by attachments. Defaults to:

```json
{
  "mini":"48x48>",
  "small":"100x100>",
  "product":"240x240>",
  "large":"600x600>"
}
```

`attachment_default_style`

A key from the list of styles from `Colibri::Config[:attachment_styles]` that is the default style for images. Defaults to the the `product` style.

`auto_capture`

Depending on whether or not Colibri is configured to "auto capture" the credit card, either a purchase or an authorize operation will be performed on the card (via the current credit card gateway).  Defaults to `false`.

`checkout_zone`

Limits the checkout to countries from a specific zone, by name. Defaults to `nil`.

`company`

Determines whether or not a field for "Company" displays on the checkout pages for shipping and billing addresses. Defaults to `false`.

`currency`

The three-letter currency code for the currency that prices will be displayed in. Defaults to "USD".

`currency_symbol_position`

The position of the symbol for a currency. Can be either `before` or `after`. Defaults to `before`.

`display_currency`

Determines whether or not a currency is displayed with a price. Defaults to `false`.

`default_country_id`

The default country's id. Defaults to 214, as this is the id for the United States within the seed data.

`default_meta_description`

The meta description to include in the `head` tag of the Colibri layout. Defaults to "Colibri demo site".

`default_meta_keywords`

The meta keywords to include in the `head` tag of the Colibri layout. Defaults to "colibri, demo".

`dismissed_colibri_alerts`

The list of alert IDs that you have dismissed.

`last_check_for_colibri_alerts`

Stores the last time that alerts were checked for. Alerts are checked for every 12 hours.

`layout`

The path to the layout of your application, relative to the `app/views` directory. Defaults to `colibri/layouts/colibri_application`. To make Colibri use your application's layout rather than Colibri's default, use this:

```ruby
Colibri.config do |config|
  config.layout = "application"
end
```

`logo`

The logo to display on your frontend. Defaults to `logo/colibri_50.png`.

`max_level_in_taxons_menu`

The number of levels to descend when viewing a taxon menu. Defaults to `1`.

`orders_per_page`

The number of orders to display on the orders listing in the admin backend. Defaults to `15`.

`prices_inc_tax`

Determines if prices are labelled as including tax or not. Defaults to `false`.

`shipment_inc_vat`

Determines if shipments should include VAT calculations. Defaults to `false`.

`shipping_instructions`

Determines if shipping instructions are requested or not when checking out. Defaults to `false`.

`show_descendents`

Determines if taxon descendants are shown when showing taxons. Defaults to `true`.

`show_only_complete_orders_by_default`

Determines if, on the admin listing screen, only completed orders should be shown. Defaults to `true`.

`show_variant_full_price`

Determines if the variant's full price or price difference from a product should be displayed on the product's show page. Defaults to `false`.

`site_name`

The name of your Colibri Store. Defaults to "Colibri Demo Site".

`site_url`

The URL for your Colibri Store. Defaults to "demo.colibricommere.com".

`tax_using_ship_address`

Determines if tax information should be based on shipping address, rather than the billing address. Defaults to `true`.

`track_inventory_levels`

Determines if inventory levels should be tracked when products are purchased at checkout. This option causes new `InventoryUnit` objects to be created when a product is bought. Defaults to `true`.

## S3 Support

To configure Colibri to upload images to S3, put these lines into `config/initializers/colibri.rb`:

```ruby
Colibri.config do |config|
  config.use_s3 = true
  config.s3_bucket = '<bucket>'
  config.s3_access_key = "<key>"
  config.s3_secret = "<secret>"
end
```

It's also a good idea to not include the `rails_root` path inside the `attachment_path` configuration option, which by default is this:

```ruby
:rails_root/public/colibri/products/:id/:style/:basename.:extension
```

To change this, add the following line underneath the `s3_secret` configuration setting:

```ruby
config.attachment_path = '/colibri/products/:id/:style/:basename.:extension'
```

If you're using the Western Europe S3 server, you will need to set two additional options inside this block:

```ruby
Colibri.config do |config|
  ...
  config.attachment_url = ":s3_eu_url"
  config.s3_host_alias = "s3-eu-west-1.amazonaws.com"
end
```

Additionally, you will need to tell `paperclip` how to construct the URLs for your images by placing this code outside the `config` block inside `config/initializers/colibri.rb`:

```ruby
Paperclip.interpolates(:s3_eu_url) do |attachment, style|
  "#{attachment.s3_protocol}://#{Colibri::Config[:s3_host_alias]}/#{attachment.bucket_name}/#{attachment.path(style).gsub(%r{^/}, "")}"
end
```
