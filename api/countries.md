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

