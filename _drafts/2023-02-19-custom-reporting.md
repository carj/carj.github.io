---
layout: post
title: Custom Reporting via the Preservica Content API
---

This article is a repost from one published on the  [Preservica Developer Blog](https://developers.preservica.com/blog/custom-reporting-via-the-preservica-content-api).


Preservica provides a REST API to allow users to query the underlying search engine. The Search engine provides access to all the indexed information within a Preservica system, this includes the standard metadata attributes such a title, description etc. and also any organization specific custom indexes added by users.

This API is available for all on-premise and Cloud Professional Edition customers and above.

This REST API is documented at [https://eu.preservica.com/api/content/documentation.html](https://eu.preservica.com/api/content/documentation.html)

In this article we will show how CSV documents can be returned by the API, CSV is a convenient format because it can be opened directly within spreadsheet software such as LibreOffice or MS Excel. Once inside a spreadsheet post processing such as filtering, sorting and charting can easily be applied to the data to create custom reports.

### Search Indexes

Before using the search API, it’s a good idea to familiarize yourself with the available indexes held by the search engine. This will give you an idea of what information can be returned by the search queries and therefore what can be reported on.

You can determine the available indexed fields within a Preservica repository by calling the

