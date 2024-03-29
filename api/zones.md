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
