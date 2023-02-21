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

You can determine the available indexed fields within a Preservica repository by calling the following endpoint.

```rest
/api/content/indexed-fields
```

You can call the API using any REST client such as [Postman](https://www.postman.com/product/api-client/), but for following examples we will use the 3rd party python library [pyPreservica](https://pypreservica.readthedocs.io/).

pyPreservica is an easy to install Python client which takes care of the authentication, xml parsing and error handling automatically.

Using pyPreservica the list of fields is [available](https://pypreservica.readthedocs.io/en/latest/content.html#indexed-fields) using:

```python
from pyPreservica import *

search = ContentAPI()

for index in search.indexed_fields():
    print(index)
```

This will print all the available indexes, The default indexes available in every system start with the xip prefix, but the list will also contain any custom indexes created when using a custom search index rules document.


```python
…
xip.format_r_Display
xip.format_r_Preservation
xip.full_text
xip.identifier
xip.is_valid_r_Display
xip.is_valid_r_Preservation
xip.order_by
xip.parent_hierarchy
xip.parent_ref
…
```

If an index on a metadata attribute is not available, its straightforward to create a new index using a custom search configuration.

You can now use these indexes to create your search query.

### Simple Reports
