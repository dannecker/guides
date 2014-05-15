---
title: Address
description: Use the Colibri Commerce storefront API to access Address data.
---

## Show

Retrieve details about a particular address:

```text
GET /api/addresses/1```

### Response

<%= headers 200 %>
<%= json(:address) %>

## Update

To update an address, make a request like this:

```text
PUT /api/addresses/1?address[firstname]=Ryan```

This request will update the `firstname` field for an address to the value of \"Ryan\"

Valid address fields are:

* firstname
* lastname
* company
* address1
* address2
* city
* zipcode
* phone
* alternative_phone
* country_id
* state_id

---
title: Checkouts
description: Use the Colibri Commerce storefront API to access Checkout data.
---

# Checkouts API

## Introduction

The checkout API functionality can be used to advance an existing order's state. Sending a `PUT` request to `/api/checkouts/ORDER_NUMBER` will advance an order's state or, failing that, report any errors.

The checkout API can also be used to create a new, empty order. Send a `POST` request to `/api/checkouts` in order to accomplish this.

The following sections will walk through creating a new order and advancing an order from its `cart` state to its `complete` state.

## Creating a blank order

To create a new, empty order, make this request:

    POST /api/checkouts

### Response

<%= headers 200 %>
<%= json :new_order %>

## Creating an order with line items

To create a new order with line items, pass line item attributes through like this:

    POST /api/checkouts?order[line_items][0][variant_id]=1&order[line_items][0][quantity]=5

<%= headers 200 %>
<%= json :new_order_with_line_items %>

## Updating an order

Updating an order using the Checkouts API will move the order automatically through its checkout flow. This is dissimilar to the Orders API, which will just update the order with the sent attributes.

To update an order with the Checkout API, you must be authenticated as the order's user, and perform a request like this:

    PUT /api/checkouts/R335381310

If you know the order's token, then you can also update the order:

    PUT /api/checkouts/R335381310?order_token=abcdef123456

Requests performed as a non-admin or non-authorized user will be met with a 401 response from this action.

## Address

To transition an order to its next step, make a request like this:

    PUT /api/checkouts/R335381310/next

### Response

This is a typical response from a transition to the `address` state:

<%= headers 200 %>
<%= json(:order_show_address_state) %>

### Failed Response

<%= headers 200 %>
<%= json(:order_failed_transition) %>

## Delivery

To advance to the next state, `delivery`, the order will first need both a shipping and billing address.

In order to update the addresses, make this request with the necessary parameters:

    PUT /api/checkouts/R335381310

As an example, here are the required address attributes and how they should be formatted:

<%= json \
  :order => {
    :bill_address_attributes => {
      :firstname  => 'John',
      :lastname   => 'Doe',
      :address1   => '7735 Old Georgetown Road',
      :city       => 'Bethesda',
      :phone      => '3014445002',
      :zipcode    => '20814',
      :state_id   => 1,
      :country_id => 1
    },

    :ship_address_attributes => {
      :firstname  => 'John',
      :lastname   => 'Doe',
      :address1   => '7735 Old Georgetown Road',
      :city       => 'Bethesda',
      :phone      => '3014445002',
      :zipcode    => '20814',
      :state_id   => 1,
      :country_id => 1
    }
  }
%>

### Response

If the parameters are valid for the request, you will see a response like this

<%= headers 200 %>
<%= json(:order_show_delivery_state) %>

Once valid address information has been submitted, the shipments and shipping rates available for this order will be returned inside a `shipments` key inside the order, as seen above.

## Payment

To advance to the next state, `payment`, you will need to select a shipping rate for each shipment for the order. These were returned when transitioning to the `delivery` step. If you need want to see them again, make the following request:

    GET /api/orders/R335381310

If the order's shipments already have shipping rates selected, you can advance it to the `payment` state by making this request:

    PUT /api/checkouts/R335381310/next

If the order doesn't have an assigned shipping rate, make the following request to select one and advance the order's state:

    PUT /api/checkouts/R366605801

With parameters such as these:

<%= json (
  {
    "order"=> {
      :shipments_attributes => [
        :selected_shipping_rate_id => 1,
        :id => 1
      ]
    }
  }) %>

***
Please ensure you select a shipping rate for each shipment in the order. In the request above, the `selected_shipping_rate_id` should be the id of the shipping rate you want to use and the `id` should be the id of the shipment you are choosing this shipping rate for.
***

### Response

<%= headers 201 %>
<%= json(:order_show_payment_state) %>

## Confirm

To advance to the next state, `confirm`, the order will need to have a payment. You can create a payment by passing in parameters such as this:

<%= json ({"order"=>
  {"payments_attributes"=>{"payment_method_id"=>"1"},
   "payment_source"=>
    {"1"=>
      {"number"=>"1",
       "month"=>"1",
       "year"=>"2017",
       "verification_value"=>"123",
       "first_name"=>"John",
       "last_name"=>"Smith"}
    }
  }
}) %>

***
The numbered key in the `payment_source` hash directly corresponds to the `payment_method_id` attribute within the `payment_attributes` key. 
***

If the order already has a payment, you can advance it to the `confirm` state by making this request:

    PUT /api/checkouts/R335381310

If the order doesn't have an assigned payment method, make the following request to setup a payment method and advance the order:

    PUT /api/checkouts/R335381310?order[payments_attributes][][payment_method_id]=1

For more information on payments, view the [payments documentation](payments).

### Response

<%= headers 200 %>
<%= json(:order_show_confirm_state) %>

## Complete

Now the order is ready to be advanced to the final state, `complete`. To accomplish this, make this request:

    PUT /api/checkouts/R335381310

### Response

<%= headers 200 %>
<%= json (:order_show_complete_state) %>
---
title: Countries
description: Use the Colibri Commerce storefront API to access Country data.
---

## Index

Retrieve a list of all countries by making this request:

```text
GET /api/countries```

Countries are paginated and can be iterated through by passing along a `page` parameter:

```text
GET /api/countries?page=2```

### Parameters

page
: The page number of country to display.

per_page
: The number of countries to return per page

### Response

<%= headers 200 %>
<%= json(:country) do |h|
{ :countries => [h],
  :count => 25,
  :pages => 5,
  :current_page => 1 }
end %>

## Search

To search for a particular country, make a request like this:

```text
GET /api/countries?q[name_cont]=united```

The searching API is provided through the Ransack gem which Colibri depends on. The `name_cont` here is called a predicate, and you can learn more about them by reading about [Predicates on the Ransack wiki](https://github.com/ernie/ransack/wiki/Basic-Searching).

The search results are paginated.

### Response

<%= headers 200 %>
<%= json(:country) do |h|
 { :countries => [h],
   :count => 25,
   :pages => 5,
   :current_page => 1 }
end %>

Results can be returned in a specific order by specifying which field to sort by when making a request.

```text
GET /api/countries?q[s]=name%20desc```

## Show

Retrieve details about a particular country:

```text
GET /api/countries/1```

### Response

<%= headers 200 %>
<%= json(:country) %>

---
title: Line Items
description: Use the Colibri Commerce storefront API to access LineItem data.
---

# Line Items API

## Create

To create a new line item, make a request like this:

    POST /api/orders/R1234567/line_items?line_item[variant_id]=1&line_item[quantity]=1

This will create a new line item representing a single item for the variant with the id of 1.

### Response

<%= headers 201 %>
<%= json(:line_item) %>

## Update

To update the information for a line item, make a request like this:

    PUT /api/orders/R1234567/line_items/1?line_item[variant_id]=1&line_item[quantity]=1

This request will update the line item with the ID of 1 for the order, updating the line item's `variant_id` to 1, and its `quantity` 1.

### Response

<%= headers 200 %>
<%= json(:line_item) %>

## Delete

To delete a line item, make a request like this:

    DELETE /api/orders/R1234567/line_items/1

### Response

<%= headers 204 %>

---
title: Orders
description: Use the Colibri Commerce storefront API to access Order data.
---

## Index

<%= admin_only %>

Retrieve a list of orders by making this request:

```text
GET /api/orders```

Orders are paginated and can be iterated through by passing along a `page` parameter:

```text
GET /api/orders?page=2```

### Parameters

page
: The page number of order to display.

per_page
: The number of orders to return per page

### Response

<%= headers 200 %>
<%= json(:order) do |h|
{ :orders => [h],
  :count => 25,
  :pages => 5,
  :current_page => 1 }
end %>

## Search

To search for a particular order, make a request like this:

```text
GET /api/orders?q[email_cont]=bob```

The searching API is provided through the Ransack gem which Colibri depends on. The `email_cont` here is called a predicate, and you can learn more about them by reading about [Predicates on the Ransack wiki](https://github.com/ernie/ransack/wiki/Basic-Searching).

The search results are paginated.

### Response

<%= headers 200 %>
<%= json(:order) do |h|
 { :orders => [h],
   :count => 25,
   :pages => 5,
   :current_page => 1 }
end %>

### Sorting results

Results can be returned in a specific order by specifying which field to sort by when making a request.

```text
GET /api/orders?q[s]=number%20desc```

It is also possible to sort results using an associated object's field.

```text
GET /api/orders?q[s]=user_name%20asc```

## Show

To view the details for a single product, make a request using that order\'s number:

```text
GET /api/orders/R123456789```

Orders through the API will only be visible to admins and the users who own
them. If a user attempts to access an order that does not belong to them, they
will be met with an authorization error.

Users may pass in the order's token in order to be authorized to view an order:

```text
GET /api/orders/R123456789?order_token=abcdef123456
```

The `order_token` parameter will work for authorizing any action for an order within Colibri's API.

### Successful Response

<%= headers 200 %>
<%= json :order_show %>

### Not Found Response

<%= not_found %>

### Authorization Failure

<%= authorization_failure %>

## Show (delivery)

When an order is in the "delivery" state, additional shipments information will be returned in the API:

<%= json(:shipment) do |h|
 { :shipments => [h] }
end %>

## Create

To create a new order through the API, make this request:

```text
POST /api/orders```

If you wish to create an order with a line item matching to a variant whose ID is \"1\" and quantity is 5, make this request:

```text
POST /api/orders?order[line_items][0][variant_id]=1&order[line_items][0][quantity]=5```

### Successful response

<%= headers 201 %>

### Failed response

<%= headers 422 %>
<%= json \
  :error => "Invalid resource. Please fix errors and try again.",
  :errors => {
    :name => ["can't be blank"],
    :price => ["can't be blank"]
  }
%>

## Update Address

To add address information to an order, please see the [checkout transitions](checkouts#checkout-transitions) section of the Checkouts guide.

## Empty

To empty an order\'s cart, make this request:

```text
PUT /api/orders/R1234567/empty```

All line items will be removed from the cart and the order\'s information will
be cleared. Inventory that was previously depleted by this order will be
repleted.
---
title: Payments
description: Use the Colibri Commerce storefront API to access Payment data.
---

# Payments API

## Index

To see details about an order's payments, make this request:

    GET /api/orders/R1234567/payments

Payments are paginated and can be iterated through by passing along a `page` parameter:

    GET /api/orders/R1234567/payments?page=2

### Parameters

page
: The page number of payment to display.

per_page
: The number of payments to return per page

### Response

<%= headers 200 %>
<%= json(:payment) do |h|
{ :payments => [h],
  :count => 2,
  :pages => 2,
  :current_page => 1 }
end %>

## Search

To search for a particular payment, make a request like this:

    GET /api/orders/R1234567/payments?q[response_code_cont]=123

The searching API is provided through the Ransack gem which Colibri depends on. The `response_code_cont` here is called a predicate, and you can learn more about them by reading about [Predicates on the Ransack wiki](https://github.com/ernie/ransack/wiki/Basic-Searching).

The search results are paginated.

### Response

<%= headers 200 %>
<%= json(:payment) do |h|
{ :payments => [h],
  :count => 2,
  :pages => 2,
  :current_page => 1 }
end %>

### Sorting results

Results can be returned in a specific order by specifying which field to sort by when making a request.

    GET /api/payments?q[s]=state%20desc

It is also possible to sort results using an associated object's field.

    GET /api/payments?q[s]=order_number%20asc

## New

In order to create a new payment, you will need to know about the available payment methods and attributes. To find these out, make this request:

    GET /api/orders/R1234567/payments/new

### Response

<%= headers 200 %>
<%= json \
  :attributes =>
  ["id", "source_type", "source_id", "amount",
   "payment_method_id", "response_code", "state",
   "avs_response", "created_at", "updated_at"],
  :payment_methods => [Colibri::Resources::PAYMENT_METHOD] %>

## Create

To create a new payment, make a request like this:

   POST /api/orders/R1234567/payments?payment[payment_method_id]=1&payment[amount]=10

### Response

<%= headers 201 %>
<%= json(:payment) %>

## Show

To get information for a particular payment, make a request like this:

   GET /api/orders/R1234567/payments/1

### Response

<%= headers 200 %>
<%= json(:payment) %>

## Authorize

To authorize a payment, make a request like this:

   PUT /api/orders/R1234567/payments/1/authorize

### Response

<%= headers 200 %>
<%= json :payment %>

### Failed Response

<%= headers 422 %>
<%= json :error => "There was a problem with the payment gateway: [text]" %>

## Capture

<%= warning "Capturing a payment is typically done shortly after authorizing the payment. If you are auto-capturing payments, you may be able to use the purchase endpoint instead." %>

To capture a payment, make a request like this:

   PUT /api/orders/R1234567/payments/1/capture

### Response

<%= headers 200 %>
<%= json :payment %>

### Failed Response

<%= headers 422 %>
<%= json :error => "There was a problem with the payment gateway: [text]" %>

## Purchase

<%= warning "Purchasing a payment is typically done only if you are not authorizing payments before-hand. If you are authorizing payments, then use the authorize and capture endpoints instead." %>

To make a purchase with a payment, make a request like this:

   PUT /api/orders/R1234567/payments/1/purchase

### Response

<%= headers 200 %>
<%= json :payment %>

### Failed Response

<%= headers 422 %>
<%= json :error => "There was a problem with the payment gateway: [text]" %>

## Void

To void a payment, make a request like this:

   PUT /api/orders/R1234567/payments/1/void

### Response

<%= headers 200 %>
<%= json :payment %>

### Failed Response

<%= headers 422 %>
<%= json :error => "There was a problem with the payment gateway: [text]" %>

## Credit

To credit a payment, make a request like this:

   PUT /api/orders/R1234567/payments/1/credit?amount=10

If the payment is over the payment's credit allowed limit, a "Credit Over Limit" response will be returned.

### Response

<%= headers 200 %>
<%= json :payment %>

### Failed Response

<%= headers 422 %>
<%= json :error => "There was a problem with the payment gateway: [text]" %>

### Credit Over Limit Response

<%= headers 422 %>
<%= json :error => "This payment can only be credited up to [amount]. Please specify an amount less than or equal to this number." %>

---
title: Product Properties
description: Use the Colibri Commerce storefront API to access ProductProperty data.
---

<%= warning "Requests to this API will only succeed if the user making them has access to the underlying products. If the user is not an admin and the product is not available yet, users will receive a 404 response from this API." %>

## Index

List

Retrieve a list of all product properties for a product by making this request:

    GET /api/products/1/product_properties

Product properties are paginated and can be iterated through by passing along a `page` parameter:

    GET /api/products/1/product_properties?page=2

### Parameters

page
: The page number of product property to display.

per_page
: The number of product properties to return per page

### Response

<%= headers 200 %>
<%= json(:product_property) do |h|
{ :product_properties => [h],
  :count => 10,
  :pages => 2,
  :current_page => 1 }
end %>

## Search

To search for a particular product property, make a request like this:

    GET /api/products/1/product_properties?q[property_name_cont]=bag

The searching API is provided through the Ransack gem which Colibri depends on. The `property_name_cont` here is called a predicate, and you can learn more about them by reading about [Predicates on the Ransack wiki](https://github.com/ernie/ransack/wiki/Basic-Searching).

The search results are paginated.

### Response

<%= headers 200 %>
<%= json(:product_property) do |h|
 { :product_properties => [h],
   :count => 10,
   :pages => 2,
   :current_page => 1 }
end %>

### Sorting results

Results can be returned in a specific order by specifying which field to sort by when making a request.

    GET /api/products/1/product_properties?q[s]=property_name%20desc

## Show

To get information about a single product property, make a request like this:

    GET /api/products/1/product_properties/1

Or you can use a property's name:

    GET /api/products/1/product_properties/size

### Response

<%= headers 200 %>
<%= json(:product_property) %>

## Create

<%= admin_only %>

To create a new product property, make a request like this:

    POST /api/products/1/product_properties?product_property[property_name]=size&product_property[value]=10

If a property with that name does not already exist, then it will automatically be created.

### Response

<%= headers 201 %>
<%= json(:product_property) %>

## Update

To update an existing product property, make a request like this:

    PUT /api/products/1/product_properties/size?product_property[value]=10

You may also use a property's id if you know it:

    PUT /api/products/1/product_properties/1?product_property[value]=10

### Response

<%= headers 200 %>
<%= json(:product_property) %>

## Delete

To delete a product property, make a request like this:

    DELETE /api/products/1/product_properties/size

<%= headers 204 %>

---
title: Products
description: Use the Colibri Commerce storefront API to access Product data.
---

## Index

List products visible to the authenticated user. If the user is not an admin, they will only be able to see products which have an `available_on` date in the past. If the user is an admin, they are able to see all products.

```text
GET /api/products```

Products are paginated and can be iterated through by passing along a `page` parameter:

```text
GET /api/products?page=2```

### Parameters

show_deleted
: **boolean** - `true` to show deleted products, `false` to hide them. Default: `false`. **Only available to users with an admin role.**

page
: The page number of products to display.

per_page
: The number of products to return per page

### Response

<%= headers 200 %>
<%= json(:product) do |h|
{ :products => [h],
  :count => 25,
  :pages => 5,
  :current_page => 1 }
end %>

## Search

To search for a particular product, make a request like this:

```text
GET /api/products?q[name_cont]=Colibri```

The searching API is provided through the Ransack gem which Colibri depends on. The `name_cont` here is called a predicate, and you can learn more about them by reading about [Predicates on the Ransack wiki](https://github.com/ernie/ransack/wiki/Basic-Searching).

The search results are paginated.

### Response

<%= headers 200 %>
<%= json(:product) do |h|
{ :products => [h],
  :count => 25,
  :pages => 5,
  :current_page => 1 }
end %>

### Sorting results

Results can be returned in a specific order by specifying which field to sort by when making a request.

```text
GET /api/products?q[s]=sku%20asc```

It is also possible to sort results using an associated object's field.

```text
GET /api/products?q[s]=shipping_category_name%20asc```

## Show

To view the details for a single product, make a request using that product\'s permalink:

```text
GET /api/products/a-product```

You may also query by the product\'s id attribute:

```text
GET /api/products/1```

Note that the API will attempt a permalink lookup before an ID lookup.

### Successful Response

<%= headers 200 %>
<%= json :product %>

### Not Found Response

<%= not_found %>

## New

You can learn about the potential attributes (required and non-required) for a product by making this request:

```text
GET /api/products/new```

### Response

<%= headers 200 %>
<%= json \
  :attributes => [
    :id, :name, :description, :price, :available_on, :permalink,
    :count_on_hand, :meta_description, :meta_keywords, :shipping_category_id, :taxon_ids
  ],
  :required_attributes => [:name, :price, :shipping_category_id]
 %>

## Create

<%= admin_only %>

To create a new product through the API, make this request with the necessary parameters:

```text
POST /api/products```

For instance, a request to create a new product called \"Headphones\" with a price of $100 would look like this:

```text
POST /api/products?product[name]=Headphones&product[price]=100&product[shipping_category_id]=1```

### Successful response

<%= headers 201 %>

### Failed response

<%= headers 422 %>
<%= json \
  :error => "Invalid resource. Please fix errors and try again.",
  :errors => {
    :name => ["can't be blank"],
    :price => ["can't be blank"],
    :shipping_category_id => ["can't be blank"]
  }
%>

## Update

<%= admin_only %>

To update a product\'s details, make this request with the necessary parameters:

```text
PUT /api/products/a-product```

For instance, to update a product\'s name, send it through like this:

```text
PUT /api/products/a-product?product[name]=Headphones```

### Successful response

<%= headers 201 %>

### Failed response

<%= headers 422 %>
<%= json \
  :error => "Invalid resource. Please fix errors and try again.",
  :errors => {
    :name => ["can't be blank"],
    :price => ["can't be blank"],
    :shipping_category_id => ["can't be blank"]
  }
%>

## Delete

<%= admin_only %>

To delete a product, make this request:

```text
DELETE /api/products/a-product```

This request, much like a typical product \"deletion\" through the admin interface, will not actually remove the record from the database. It simply sets the `deleted_at` field to the current time on the product, as well as all of that product\'s variants.

<%= headers 204 %>
---
title: Return Authorizations
description: Use the Colibri Commerce storefront API to access ReturnAuthorization data.
---

# Return Authorizations API

<%= admin_only %>

## Index

To list all return authorizations for an order, make a request like this:

    GET /api/orders/R1234567/return_authorizations

Return authorizations are paginated and can be iterated through by passing along a `page` parameter:

    GET /api/orders/R1234567/return_authorizations?page=2

### Parameters

page
: The page number of return authorization to display.

per_page
: The number of return authorizations to return per page

### Response

<%= headers 200 %>
<%= json(:return_authorization) do |h|
{ :return_authorizations => [h],
  :count => 2,
  :pages => 1,
  :current_page => 1 }
end %>

## Search

To search for a particular return authorization, make a request like this:

    GET /api/orders/R1234567/return_authorizations?q[reason_cont]=damage

The searching API is provided through the Ransack gem which Colibri depends on. The `reason_cont` here is called a predicate, and you can learn more about them by reading about [Predicates on the Ransack wiki](https://github.com/ernie/ransack/wiki/Basic-Searching).

The search results are paginated.

### Sorting results

Results can be returned in a specific order by specifying which field to sort by when making a request.

    GET /api/orders/R1234567/return_authorizations?q[s]=amount%20asc

### Response

<%= headers 200 %>
<%= json(:return_authorization) do |h|
 { :return_authorizations => [h],
   :count => 1,
   :pages => 1,
   :current_page => 1 }
end %>

## Show

To get information for a single return authorization, make a request like this:

     GET /api/orders/R1234567/return_authorizations/1

### Response

<%= headers 200 %>
<%= json(:return_authorization) %>

## Create

<%= admin_only %>

To create a return authorization, make a request like this:

     POST /api/orders/R1234567/return_authorizations

For instance, if you want to create a return authorization with a number, make
this request:

     POST /api/orders/R1234567/return_authorizations?return_authorization[number]=123456

### Response

<%= headers 201 %>
<%= json(:return_authorization) %>

## Update

<%= admin_only %>

To update a return authorization, make a request like this:

     PUT /api/orders/R1234567/return_authorizations/1

For instance, to update a return authorization's number, make this request:

     PUT /api/orders/R1234567/return_authorizations/1?return_authorization[number]=123456

### Response

<%= headers 200 %>
<%= json(:return_authorization) %>

## Delete

<%= admin_only %>

To delete a return authorization, make a request like this:

    DELETE /api/orders/R1234567/return_authorizations/1

### Response

<%= headers 204 %>

---
title: Shipments
description: Use the Colibri Commerce storefront API to access Shipment data.
---

# Shipments API

## Create

<%= admin_only %>

The following attributes are required when creating a shipment:

- order_id
- stock_location_id
- variant_id

To create a shipment, make a request like this:

```text
POST /api/shipments?shipment[order_id]=R1234567```

The `order_id` is the number of the order to create a shipment for and is provided as part of the URL string as shown above. The shipment will be created at the selected stock location and include the variant selected.

Assuming in this instance that you want to create a shipment with a stock_location_id of `1` and a variant_id of `10` for order `R1234567`, send through the parameters like this:

<%= json \
  :order_id => 123456,
  :stock_location_id => 1,
  :variant_id => 10
 %>

### Response

<%= headers 200 %>
<%= json(:shipment) %>

## Update

<%= admin_only %>

To update shipment information, make a request like this:

```text
PUT /api/shipments/H123456789?shipment[tracking]=TRK9000```

### Parameters

unlock
: When set to `yes`, the shipment's adjustment will be recalculated.

To update order ship method inspect order/shipments/shipping_rates for available shipping_rate_id values and use following api call:

    PUT /api/shipments/H123456789?shipment[selected_shipping_rate_id]=162&shipment[unlock]=yes

### Response

<%= headers 200 %>
<%= json(:shipment) %>

## Ready

<%= admin_only %>

To mark a shipment as ready, make a request like this:

    PUT /api/shipments/H123456789/ready

You may choose to update shipment attributes with this request as well:

    PUT /api/shipments/H123456789/ready?shipment[number]=1234567

### Response

<%= headers 200 %>
<%= json(:shipment) %>

## Ship

<%= admin_only %>

To mark a shipment as shipped, make a request like this:

    PUT /api/shipments/H123456789/ship

You may choose to update shipment attributes with this request as well:

    PUT /api/shipments/H123456789/ship?shipment[number]=1234567

### Response

<%= headers 200 %>
<%= json(:shipment) %>

## Add Variant

<%= admin_only %>

To add a variant to a shipment, make a request like this:

    PUT /api/shipments/H123456789/add?variant_id=1&quantity=1

### Response

<%= headers 200 %>
<%= json(:shipment) %>

## Remove Variant

<%= admin_only %>

To remove a variant from a shipment, make a request like this:

    PUT /api/shipments/H123456789/remove?variant_id=1&quantity=1

### Response

<%= headers 200 %>
<%= json(:shipment) %>
---
title: States
description: Use the Colibri Commerce storefront API to access State data.
---

## Index

To get a list of states within Colibri, make a request like this:

```text
GET /api/states```

States are paginated and can be iterated through by passing along a `page`
parameter:

```text
GET /api/states?page=2```

As well as a `per_page` parameter to control how many results will be returned:

```text
GET /api/states?per_page=100```

You can scope the states by country by passing along a `country_id` parameter
too:

```text
GET /api/states?country_id=1```

### Response

<%= headers 200 %>
<%= json(:state) do |h|
{ :states => [h],
  :count => 25,
  :pages => 5,
  :current_page => 1 }
end %>

## Show

To find out about a single state, make a request like this:

```text
GET /api/states/1```

### Response

<%= headers 200 %>
<%= json(:state) %>
---
title: Stock Items
description: Use the Colibri Commerce storefront API to access StockItem data.
---

## Index

<%= admin_only %>

To return a paginated list of all stock items for a stock location, make this request, passing the stock location id you wish to see stock items for:

```text
GET /api/stock_locations/1/stock_items```

### Parameters

page
: The page number of stock items to display.

per_page
: The number of stock items to return per page

### Response

<%= headers 200 %>
<%= json(:stock_item) do |h|
{ :stock_items => [h],
  :count => 25,
  :pages => 5,
  :current_page => 1 }
end %>

## Search

<%= admin_only %>

To search for a particular stock item, make a request like this:

```text
GET /api/stock_locations/1/stock_items?q[variant_id_eq]=10```

The searching API is provided through the Ransack gem which Colibri depends on. The `variant_id_eq` here is called a predicate, and you can learn more about them by reading about [Predicates on the Ransack wiki](https://github.com/ernie/ransack/wiki/Basic-Searching).

The search results are paginated.

### Response

<%= headers 200 %>
<%= json(:stock_item) do |h|
 { :stock_items => [h],
   :count => 25,
   :pages => 5,
   :current_page => 1 }
end %>

### Sorting results

Results can be returned in a specific order by specifying which field to sort by when making a request.

```text
GET /api/stock_locations/1/stock_items?q[s]=variant_id%20asc```

## Show

<%= admin_only %>

To view the details for a single stock item, make a request using that stock item's id, along with its `stock_location_id`:

```text
GET /api/stock_locations/1/stock_items/2```

### Successful Response

<%= headers 200 %>
<%= json :stock_item %>

### Not Found Response

<%= not_found %>

## Create

<%= admin_only %>

To create a new stock item for a stock location, make this request with the necessary parameters:

```text
POST /api/stock_locations/1/stock_items```

For instance, a request to create a new stock item with a count_on_hand of 10 and a variant_id of 1 would look like this::

<%= json \
  :stock_item => {
    :count_on_hand => "10",
    :variant_id => "1",
    :backorderable => "true"
  } %>

### Successful response

<%= headers 201 %>
<%= json(:stock_item) %>

### Failed response

<%= headers 422 %>
<%= json \
  :error => "Invalid resource. Please fix errors and try again.",
  :errors => {
  }
%>

## Update

<%= admin_only %>

Note that using this endpoint, count_on_hand is APPENDED to its current value.

Sending a request with a negative count_on_hand will subtract the current value.

To force a value for count_on_hand, include force: true in your request, this will replace the current
value as it's stored in the database.

To update a stock item's details, make this request with the necessary parameters.

```text
PUT /api/stock_locations/1/stock_items/2```

For instance, to update a stock item's count_on_hand, send it through like this:

<%= json \
  :stock_item => {
    :count_on_hand => "30",
  } %>

Or alternatively with the force attribute to replace the current count_on_hand with a new value:

<%= json \
  :stock_item => {
    :count_on_hand => "30",
    :force => true,
  } %>

### Successful response

<%= headers 201 %>
<%= json(:stock_item) %>

### Failed response

<%= headers 422 %>
<%= json \
  :error => "Invalid resource. Please fix errors and try again.",
  :errors => {
  }
%>

## Delete

<%= admin_only %>

To delete a stock item, make this request:

```text
DELETE /api/stock_locations/1/stock_items/2```

### Response

<%= headers 204 %>
---
title: Stock Locations
description: Use the Colibri Commerce storefront API to access StockLocation data.
---

## Index

<%= admin_only %>

To get a list of stock locations, make this request:

```text
GET /api/stock_locations```

Stock locations are paginated and can be iterated through by passing along a `page` parameter:

```text
GET /api/stock_locations?page=2```

### Parameters

page
: The page number of stock location to display.

per_page
: The number of stock locations to return per page

### Response

<%= headers 200 %>
<%= json(:stock_location) do |h|
{ :stock_locations => [h],
  :count => 5,
  :pages => 1,
  :current_page => 1 }
end %>

## Search

<%= admin_only %>

To search for a particular stock location, make a request like this:

```text
GET /api/stock_locations?q[name_cont]=default```

The searching API is provided through the Ransack gem which Colibri depends on. The `name_cont` here is called a predicate, and you can learn more about them by reading about [Predicates on the Ransack wiki](https://github.com/ernie/ransack/wiki/Basic-Searching).

The search results are paginated.

### Response

<%= headers 200 %>
<%= json(:stock_location) do |h|
{ :stock_locations => [h],
  :count => 5,
  :pages => 1,
  :current_page => 1 }
end %>

## Show

<%= admin_only %>

To get information for a single stock location, make this request:

```text
GET /api/stock_locations/1```

### Response

<%= headers 200 %>
<%= json(:stock_location) %>

## Create

<%= admin_only %>

To create a stock location, make a request like this:

```text
POST /api/stock_locations```

Assuming in this instance that you want to create a stock location with a name of `East Coast`, send through the parameters like this:

<%= json \
  :stock_location => {
    :name => "East Coast",
    :action => "true"
  } %>

### Response

<%= headers 201 %>
<%= json(:stock_location) %>

## Update

<%= admin_only %>

To update a stock location, make a request like this:

```text
PUT /api/stock_locations/1```

To update stock location information, use parameters like this:

<%= json \
  :stock_location => {
    :name => "North Pole",
    :action => "false"
  } %>

### Response

<%= headers 200 %>
<%= json(:stock_location) %>

## Delete

<%= admin_only %>

To delete a stock location, make a request like this:

```text
DELETE /api/stock_locations/1```

This request will also delete any related `stock item` records.

### Response

<%= headers 204 %>
---
title: Stock Movements
description: Use the Colibri Commerce storefront API to access StockMovement data.
---

## Index

<%= admin_only %>

To return a paginated list of all stock movements for a stock location, make this request, passing the stock location id you wish to see stock items for:

```text
GET /api/stock_locations/1/stock_movements```

### Parameters

page
: The page number of stock movements to display.

per_page
: The number of stock movements to return per page

### Response

<%= headers 200 %>
<%= json(:stock_movement) do |h|
{ :stock_movements => [h],
  :count => 25,
  :pages => 5,
  :current_page => 1 }
end %>

## Search

<%= admin_only %>

To search for a particular stock movement, make a request like this:

```text
GET /api/stock_locations/1/stock_movements?q[quantity_eq]=10```

The searching API is provided through the Ransack gem which Colibri depends on. The `quantity_eq` here is called a predicate, and you can learn more about them by reading about [Predicates on the Ransack wiki](https://github.com/ernie/ransack/wiki/Basic-Searching).

The search results are paginated.

### Response

<%= headers 200 %>
<%= json(:stock_movement) do |h|
 { :stock_movements => [h],
   :count => 25,
   :pages => 5,
   :current_page => 1 }
end %>

### Sorting results

Results can be returned in a specific order by specifying which field to sort by when making a request.

```text
GET /api/stock_locations/1/stock_movements?q[s]=quantity%20asc```

## Show

<%= admin_only %>

To view the details for a single stock movement, make a request using that stock movement's id, along with its `stock_location_id`:

```text
GET /api/stock_locations/1/stock_movements/1```

### Successful Response

<%= headers 200 %>
<%= json :stock_movement %>

### Not Found Response

<%= not_found %>

## Create

<%= admin_only %>

To create a new stock movement for a stock location, make this request with the necessary parameters:

```text
POST /api/stock_locations/1/stock_movements```

For instance, a request to create a new stock movement with a quantity of 10, the action set to received, and a stock_item_id of 1 would look like this::

<%= json \
  :stock_movement => {
    :quantity => "10",
    :stock_item_id => "1",
    :action => "received"
  } %>

### Successful response

<%= headers 201 %>
<%= json(:stock_movement) %>

### Failed response

<%= headers 422 %>
<%= json \
  :error => "Invalid resource. Please fix errors and try again.",
  :errors => {
  }
%>

## Update

<%= admin_only %>

To update a stock movement's details, make this request with the necessary parameters:

```text
PUT /api/stock_locations/1/stock_movements/1```

For instance, to update a stock movement's quantity, send it through like this:

<%= json \
  :stock_movement => {
    :quantity => "30",
  } %>

### Successful response

<%= headers 201 %>
<%= json(:stock_movement) %>

### Failed response

<%= headers 422 %>
<%= json \
  :error => "Invalid resource. Please fix errors and try again.",
  :errors => {
  }
%>

## Delete

<%= admin_only %>

To delete a stock movement, make this request:

```text
DELETE /api/stock_locations/1/stock_movement/1```

### Response

<%= headers 204 %>
---
title: Summary
---

## Overview

Colibri currently supports RESTful access to the resources listed in the sidebar
on the right &raquo;

This API was built using the great [Rabl](https://github.com/nesquena/rabl) gem.
Please consult its documentation if you wish to understand how the templates use
it to return data.

This API conforms to a set of [rules](#rules).

### JSON Data

Developers communicate with the Colibri API using the [JSON](http://www.json.org) data format. Requests for data are communicated in the standard manner using the HTTP protocol.

### Making an API Call

You will need an authentication token to access the API. These keys can be generated on the user edit screen within the admin interface. To make a request to the API, pass a `X-Colibri-Token` header along with the request:

```bash
$ curl --header "X-Colibri-Token: YOUR_KEY_HERE" http://example.com/api/products.json```


Alternatively, you may also pass through the token as a parameter in the request if a header just won't suit your purposes (i.e. JavaScript console debugging).

```bash
$ curl http://example.com/api/products.json?token=YOUR_KEY_HERE```

The token allows the request to assume the same level of permissions as the actual user to whom the token belongs.

### Error Messages

You may encounter the follow error messages when using the API.

#### Not Found

<%= not_found %>

#### Authorization Failure

<%= authorization_failure %>

#### Invalid API Key

<%= headers 401 %>
<%= json(:error => "Invalid API key ([key]) specified.") %>

## Rules

The following are some simple rules that all Colibri API endpoints comply with.

1. All successful requests for the API will return a status of 200.
2. Successful create and update requests will result in a status of 201 and 200 respectively.
3. Both create and update requests will return Colibri\'s representation of the data upon success.
4. If a create or update request fails, a status code of 422 will be returned, with a hash containing an \"error\" key, and an \"errors\" key. The errors value will contain all ActiveRecord validation errors encountered when saving this record.
5. Delete requests will return status of 200, and no content.
6. Requests that list collections, such as /api/products will return a limited result set back.
7. Requests that list collections can be paginated through by passing a page parameter that is a number greater than 0.
8. If a resource can not be found, the API will return a status of 404.
9. Unauthorized requests will be met with a 401 response.

## Customizing Responses

If you wish to customize the responses from the API, you can do this in one of
two ways: overriding the template, or providing a custom template.

### Overriding template

Overriding a template for the API should be done if you want to *always* provide
a custom response for an API endpoint. Template loading in Rails will attempt to
look up a template within your application's view paths first. If it isn't
available there, then it will fallback to looking within the other engine's view
paths, eventually finding its way to the API engine.

You can use this to your advantage and define a view template within your
application that exists at the same path as a template within the API engine.
For instance, if you place a template in your application at
`app/views/colibri/api/products/show.v1.rabl`, it will take precedence over the
template within the API engine.

This is the method we would recommend to *completely* override an API response.

### Custom template

If you don't want to always override the response for an API controller, you can
customize it in another way by creating an alternative template to use for some
API responses.

To do this, create a template under the view directory of your targetted
resource. For instance, if you wanted to customize a response for one of the
actions within the `ProductsController` of the API, you would place the template
at `app/views/colibri/api/products`. The template must be given a unique name that
won't conflict with any other templates; you could call it `small_show` for
instance.

If you were to take this route, the new template file's path would be
`app/views/colibri/api/products/small_show.v1.rabl`. The `v1` part of the filename
indicates that its a response for version 1 of the API, and the `rabl` on the
end is the markup language used.

To use this new template for your API response, simply pass the `template`
parameter along with the request:
`http://example.com/api/products/1?template=small_show`. The API component of
Colibri will then detect this parameter, find the template, and then use this to
render the response.
---
title: Taxonomies
description: Use the Colibri Commerce storefront API to access Taxonomy data.
---

## Index

To get a list of all the taxonomies, including their root nodes and the
immediate children for the root node, make a request like this:

```text
GET /api/taxonomies```

### Parameters

page
: The page number of taxonomy to display.

per_page
: The number of taxonomies to return per page

### Response

<%= headers 200 %>
<%= json(:taxonomy) do |h|
{ :taxonomies => [h],
  :count => 25,
  :pages => 5,
  :current_page => 1 }
end %>

## Search

To search for a particular taxonomy, make a request like this:

```text
GET /api/taxonomies?q[name_cont]=brand```

The searching API is provided through the Ransack gem which Colibri depends on. The `name_cont` here is called a predicate, and you can learn more about them by reading about [Predicates on the Ransack wiki](https://github.com/ernie/ransack/wiki/Basic-Searching).

The search results are paginated.

### Response

<%= headers 200 %>
<%= json(:taxonomy) do |h|
 { :taxonomies => [h],
   :count => 5,
   :pages => 2,
   :current_page => 1 }
end %>

### Sorting results

Results can be returned in a specific order by specifying which field to sort by when making a request.

```text
GET /api/taxonomies?q[s]=name%20asc```

It is also possible to sort results using an associated object's field.

```text
GET /api/taxonomies?q[s]=root_name%20desc```

## Show

To get information for a single taxonomy, including its root node and the immediate children of the root node, make a request like this:

```text
GET /api/taxonomies/1```

### Response

<%= headers 200 %>
<%= json(:taxonomy) %>

## Create

<%= admin_only %>

To create a taxonomy, make a request like this:

```text
POST /api/taxonomies```

For instance, if you want to create a taxonomy with the name \"Brands\", make
this request:

```text
POST /api/taxonomies?taxonomy[name]=Brand```

If you\'re creating a taxonomy without a root taxon, a root taxon will automatically be
created for you with the same name as the taxon.

### Response

<%= headers 201 %>
<%= json(:new_taxonomy) %>

## Update

<%= admin_only %>

To update a taxonomy, make a request like this:

```text
PUT /api/taxonomies/1```

For instance, to update a taxonomy\'s name, make this request:

```text
PUT /api/taxonomies/1?taxonomy[name]=Brand```

### Response

<%= headers 200 %>
<%= json(:taxonomy) %>

## Delete

<%= admin_only %>

To delete a taxonomy, make a request like this:

```text
DELETE /api/taxonomies/1```

### Response

<%= headers 204 %>

## List taxons

To get a list for all taxons underneath the root taxon for a taxonomy (and their immediate children) for a taxonomy, make this request:

    GET /api/taxonomies/1/taxons

### Response

<%= headers 200 %>
<%= json(:taxon_with_children) { |h| [h] } %>

## A single taxon

To see information about a taxon and its immediate children, make a request
like this:

    GET /api/taxonomies/1/taxons/1

### Response

<%= headers 200 %>
<%= json(:taxon_with_children) %>


## Taxon Create

<%= admin_only %>

To create a taxon, make a request like this:

    POST /api/taxonomies/1/taxons

To create a new taxon with the name "Brands", make this request:

    POST /api/taxonomies/1/taxons?taxon[name]=Brands

### Response

<%= headers 201 %>
<%= json(:taxon_without_children) %>


## Taxon Update

<%= admin_only %>

To update a taxon, make a request like this:

    PUT /api/taxonomies/1/taxons/1

For example, to update the taxon's name to "Brand", make this request:

    PUT /api/taxonomies/1/taxons/1?taxon[name]=Brand

### Response

<%= headers 200 %>
<%= json(:taxon_with_children) %>

## Taxon Delete

<%= admin_only %>

To delete a taxon, make a request like this:

    DELETE /api/taxonomies/1/taxons/1

<%= warning "This will cause all child taxons to be deleted as well." %>

### Response

<%= headers 204 %>
---
title: Users
description: Use the Colibri Commerce storefront API to access User data.
---

List users visible to the authenticated user. If the user is not an admin,
they will only be able to see their own user, unless they have custom
permissions to see other users. If the user is an admin then they can see all
users.

```text
GET /api/users```

Users are paginated and can be iterated through by passing along a `page`
parameter:

```text
GET /api/users?page=2```

### Response

<%= headers 200 %>
<%= json(:user) do |h|
    { :users => [h], :count => 25, :pages => 5, :current_page => 1 }
end %>

## A single user

To view the details for a single user, make a request using that user\'s
id:

```text
GET /api/users/1```

### Successful Response

<%= headers 200 %> <%= json :user %>

### Not Found Response

<%= not_found %>

## Pre-creation of a user

You can learn about the potential attributes (required and non-required) for a
user by making this request:

```text GET /api/users/new```

### Response

<%= headers 200 %>
<%= json :attributes => ["<attribute1>", "<attribute2>"], :required_attributes => [] %>

## Creating a new new

<%= admin_only %>

To create a new user through the API, make this request with the necessary
parameters:

```text
POST /api/users```

For instance, a request to create a new user with the email
\"colibri@example.com\" and password \"password\" would look like this:

```text
POST /api/users?user[email]=colibri@example.com&user[password]=password```

### Successful response

<%= headers 201 %>

### Failed response

<%= headers 422 %>
<%= json :error => "Invalid resource. Please fix errors and try again.",
         :errors => { :email => ["can't be blank"] } %>

## Updating a user

<%= admin_only %>

To update a user\'s details, make this request with the necessary parameters:

```text
PUT /api/users/1```

For instance, to update a user\'s password, send it through like this:

```text PUT /api/users/1?user[password]=password```

### Successful response

<%= headers 201 %>

### Failed response

<%= headers 422 %>
<%= json :error => "Invalid resource. Please fix errors and try again.",
         :errors => { :email => ["can't be blank"] } %>

## Deleting a user

<%= admin_only %>

To delete a user, make this request:

```text
DELETE /api/users/1```

### Response

<%= headers 204 %>

---
title: Variants
description: Use the Colibri Commerce storefront API to access Variant data.
---

## Index

To return a paginated list of all variants within the store, make this request:

```text
GET /api/variants```

You can limit this to showing the variants for a particular product by passing through a product's permalink:

```text
GET /api/products/ruby-on-rails-tote/variants```

or

```text
GET /api/variants?product_id=ruby-on-rails-tote```

### Parameters

show_deleted
: **boolean** - `true` to show deleted variants, `false` to hide them. Default: `false`. **Only available to users with an admin role.**

page
: The page number of variants to display.

per_page
: The number of variants to return per page

### Response

<%= headers 200 %>
<%= json(:variant) do |h|
{ :variants => [h],
  :count => 25,
  :pages => 5,
  :current_page => 1 }
end %>

## Search

To search for a particular variant, make a request like this:

```text
GET /api/variants?q[sku_cont]=foo```

You can limit this to showing the variants for a particular product by passing through a product id:

```text
GET /api/products/ruby-on-rails-tote/variants?q[sku_cont]=foo```

or

```text
GET /api/variants?product_id=ruby-on-rails-tote&q[sku_cont]=foo```


The searching API is provided through the Ransack gem which Colibri depends on. The `sku_cont` here is called a predicate, and you can learn more about them by reading about [Predicates on the Ransack wiki](https://github.com/ernie/ransack/wiki/Basic-Searching).

The search results are paginated.

### Response

<%= headers 200 %>
<%= json(:variant) do |h|
 { :variants => [h],
   :count => 25,
   :pages => 5,
   :current_page => 1 }
end %>

### Sorting results

Results can be returned in a specific order by specifying which field to sort by when making a request.

```text
GET /api/variants?q[s]=price%20asc```

It is also possible to sort results using an associated object's field.

```text
GET /api/variants?q[s]=product_name%20asc```

## Show

To view the details for a single variant, make a request using that variant\'s id, along with the product's permalink as its `product_id`:

```text
GET /api/products/ruby-on-rails-tote/variants/1```

Or:

```text
GET /api/variants/1?product_id=ruby-on-rails-tote```

### Successful Response

<%= headers 200 %>
<%= json :variant %>

### Not Found Response

<%= not_found %>

## New

You can learn about the potential attributes (required and non-required) for a variant by making this request:

```text
GET /api/products/ruby-on-rails-tote/variants/new```

### Response

<%= headers 200 %>
<%= json \
  :attributes => [
    :id, :name, :count_on_hand, :sku, :price, :weight, :height,
    :width, :depth, :is_master, :cost_price, :permalink
  ],
  :required_attributes => []
 %>

## Create

<%= admin_only %>

To create a new variant for a product, make this request with the necessary parameters:

```text
POST /api/products/ruby-on-rails-tote/variants```

For instance, a request to create a new variant with a SKU of 12345 and a price of 19.99 would look like this::

```text
POST /api/products/ruby-on-rails-tote/variants/?variant[sku]=12345&variant[price]=19.99```

### Successful response

<%= headers 201 %>

### Failed response

<%= headers 422 %>
<%= json \
  :error => "Invalid resource. Please fix errors and try again.",
  :errors => {
  }
%>

## Update

<%= admin_only %>

To update a variant\'s details, make this request with the necessary parameters:

```text
PUT /api/products/ruby-on-rails-tote/variants/2```

For instance, to update a variant\'s SKU, send it through like this:

```text
PUT /api/products/ruby-on-rails-tote/variants/2?variant[sku]=12345```

### Successful response

<%= headers 201 %>

### Failed response

<%= headers 422 %>
<%= json \
  :error => "Invalid resource. Please fix errors and try again.",
  :errors => {
  }
%>

## Delete

<%= admin_only %>

To delete a variant, make this request:

```text
DELETE /api/products/ruby-on-rails-tote/variants/2```

This request, much like a typical variant \"deletion\" through the admin interface, will not actually remove the record from the database. It simply sets the `deleted_at` field to the current time on the variant.

<%= headers 204 %>

---
title: Zones
description: Use the Colibri Commerce storefront API to access Zone data.
---

## Index

To get a list of zones, make this request:

```text
GET /api/zones```

Zones are paginated and can be iterated through by passing along a `page` parameter:

```text
GET /api/zones?page=2```

### Parameters

page
: The page number of zone to display.

per_page
: The number of zones to return per page

### Response

<%= headers 200 %>
<%= json(:zone) do |h|
{ :zones => [h],
  :count => 25,
  :pages => 5,
  :current_page => 1 }
end %>

## Search

To search for a particular zone, make a request like this:

```text
GET /api/zones?q[name_cont]=north```

The searching API is provided through the Ransack gem which Colibri depends on. The `name_cont` here is called a predicate, and you can learn more about them by reading about [Predicates on the Ransack wiki](https://github.com/ernie/ransack/wiki/Basic-Searching).

The search results are paginated.

### Response

<%= headers 200 %>
<%= json(:zone) do |h|
 { :zones => [h],
   :count => 25,
   :pages => 5,
   :current_page => 1 }
end %>

### Sorting results

Results can be returned in a specific order by specifying which field to sort by when making a request.

```text
GET /api/zones?q[s]=name%20desc```

## Show

To get information for a single zone, make this request:

```text
GET /api/zones/1```

### Response

<%= headers 200 %>
<%= json(:zone) %>

## Create

<%= admin_only %>

To create a zone, make a request like this:

```text
POST /api/zones```

Assuming in this instance that you want to create a zone containing
a zone member which is a `Colibri::Country` record with the `id` attribute of 1, send through the parameters like this:

<%= json \
  :zone => {
    :name => "North Pole",
    :zone_members => [
      {
        :zoneable_type => "Colibri::Country",
        :zoneable_id => 1
      }
    ]
  } %>

### Response

<%= headers 201 %>
<%= json(:zone) %>

## Update

<%= admin_only %>

To update a zone, make a request like this:

```text
PUT /api/zones/1```

To update zone and zone member information, use parameters like this:

<%= json \
  :id => 1,
  :zone => {
    :name => "North Pole",
    :zone_members => [
      {
        :zoneable_type => "Colibri::Country",
        :zoneable_id => 1
      }
    ]
  } %>

### Response

<%= headers 200 %>
<%= json(:zone) %>

## Delete

<%= admin_only %>

To delete a zone, make a request like this:

```text
DELETE /api/zones/1```

This request will also delete any related `zone_member` records.

### Response

<%= headers 204 %>
