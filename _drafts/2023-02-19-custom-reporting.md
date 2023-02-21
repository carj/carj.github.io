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

Using the raw REST API you would first create a JSON document containing the search terms and POST it to the endpoint.

```rest
/api/content/search
```

The JSON document syntax is described in the [Swagger documentation](https://eu.preservica.com/api/content/documentation.html#/%2F/post_search).

Again to simplify the process we will use the pyPreservica client which creates the JSON document for you.

To create a report using the basic “google” type search which searches across all the indexes we use the


```python
simple_search_csv() 
```

method.

You can include any query term to search on, the special % character matches on everything in the repository.

This function will search the Preservica repository for the term supplied in the query argument across all indexes and save the results into a UTF-8 CSV file which can be opened in MS Excel.

For example, the following script will create a CSV file containing everything in the repository.



```python
from pyPreservica import *

search = ContentAPI()
search.simple_search_csv(query="%", csv_file="results.csv")
```

By default the spreadsheet columns correspond to the following indexes

- xip.reference The entity reference of the asset or folder
- xip.title The title of the asset or folder
- xip.description The description of the asset or folder
- xip.document_type The type of entity, e.g, “IO” for asset and “SO” for folder
- xip.parent_ref The entity reference of the parent folder
- xip.security_descriptor The security tag

Once opened in Excel, then you can use the standard tools for filtering and sorting the tabulated data.


