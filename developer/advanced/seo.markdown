---
title: "Seo Considerations"
section: advanced
---

Search Engine Optimization is an important area to address when
implementing and developing an ecommerce solution to ensure competitive
search engine performance. The following guide outlines current Colibri
Search Engine Optimization features and future optimization development
possibilities.


### Existing Search Engine Optimization Features

Chapter 1 contains a description of the work that has been completed to
address common search engine optimization issues.

#### Relevant, Meaningful URLs

The helper method `seo_url(taxon)` yields SEO friendly URLs such as [demo.usoft.com.ua/colibri/products/xm-direct2-interface-adapter](http://demo.usoft.com.ua/colibri/products/xm-direct2-interface-adapter) and [demo.usoft.com.ua/colibri/t/categories/headphones](http://demo.usoft.com.ua/colibri/t/categories/headphones).
Each controller is configured to serve the content using these keyword-relevant, meaningful URLs.

#### On Page Keyword Targeting

Several enhancements have been made to improve on-page keyword
targeting. The admin interface provides the ability to manage meta
descriptions and meta keywords at the product level. Additionally, H1
tags are used throughout the site for product and taxonomy names. The
ease of extension development and layout changes allows you to target
keywords throughout the site.

#### Clean Content

Colibri uses Skeleton, a responsive CSS framework that allows clean HTML
that also responds well to any screen size. Having clean HTML with
minimal inline JavaScript and CSS is considered to be a factor in search
engine optimization.

#### On Site Performance Optimization

Colibri has been configured to serve one CSS and one JavaScript file on
every page (excluding extension inclusions). Minimizing HTTP requests is
considered an important factor in search engine optimization as the
server performance is an important influence in the search engine crawl
behavior for a site.

#### Google Analytics integration

Google Analytics has been integrated into the Colibri core and can be
managed from the "Analytics Trackers" section of the admin. Google
Analytics is not included on your store if this preference is not set.
The Google Analytics setup includes e-commerce conversion tracking.

### Planned Search Engine Optimization Features

Although several common search engine optimization issues have been
addressed, we are always looking for the new best practices in SEO.
Contributions to address issues will be very welcome. Visit the
[contributing to colibri section](contributing.html) to learn
more about contributing.

#### Product and Taxonomy Page Title Enhancements

Page titles are an important part of search engine optimization and
should be meaningful and relevant to the page content. There are a few
configuration settings for getting the best page titles possible. "Site
Name" will appear in the beginning of each of your titles. When
possible, we assign an appropriate title after that (product name, taxon
name, etc), but when we can't do that, we use the "Default Seo Title".

#### Alt Attribute on Product Images

The alt attribute on product images currently pulls data from product
titles or the image filename. Enhancing the image alt tag can improve
image search performance.

#### Known Duplicate Content Issues

In the Colibri demo, it is a known issue that
[demo.usoft.com.ua/colibri](http://demo.usoft.com.ua/colibri/) contains
duplicate content to
[demo.usoft.com.ua/colibri/products](http://demo.usoft.com.ua/colibri/products).
Duplicate content can be a detriment to search engine performance as
external links are divided among duplicate content pages. As a result,
duplicate content pages may not only not be excluded from the main
search engine index, but pages may also rank poorly in comparison to
other sites where all external links go to one non-duplicated page.

#### Integration of Content Management System or Content

There has been quite a bit of interest in development of [CMS
integration into
Colibri](http://groups.google.com/group/colibri-user/search?q=cms). Having
good content is an important part of search engine optimization, as it
not only can improve on page keyword targeting, but it also can improve
the popularity of a site which can in turn improve search engine
optimization.

#### Tool Integration

In addition to integration of Google Analytics, several other tools can
be implemented for SEO purposes such as Bing Webmaster Tools, Google
Webmaster Tools and Quantcast. Social media optimization tools such as
Pinterest, Reddit, Digg, Delicious, Facebook, Google+ and Twitter may
also be integrated to improve social networking site performance.

### Colibri SEO Extensions

The following list shows extensions that can improve search engine
performance. Refer to the GitHub README for developer notes.

-   [Static Content Management](https://github.com/colibri/colibri_static_content)
-   [Colibri Sitemap Generation](https://github.com/romul/colibri_dynamic_sitemaps)
-   [Product Reviews](https://github.com/colibri/colibri_reviews)

### External Search Engine Optimization Efforts

Colibri cannot control factors such as external links, quality of external
links, server performance and capabilities. These areas should not be
ignored in implementation of search engine optimization efforts.
