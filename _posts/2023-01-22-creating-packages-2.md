---
layout: post
title: Customising Packages using the pyPreservica Python SDK
---

In the previous [article](https://carj.github.io/2023/01/21/creating-packages/) we looked at how to create packages using package creation functions available in the  pyPreservica API.

* `simple_asset_package`
* `complex_asset_package`
* `generic_asset_package`

Each of these functions can be used to create packages of varying complexity and structure.

These functions provide sensible default values for the submission, but all the defaults can be overwitten.

### Identifiers

Preservica Assets can contain multiple 3rd party external identifiers. Identifiers are key value pairs, the key is the identifier name or type.
Identifier values do not have to be unique.

The following shows a Preservica Asset with some common identifiers.

![indentifiers](/public/images/identifiers.png)

We can add identifiers to submission packages by creating a Python dictionary of identifier keys and values and passing it to one of the package creation methods.

For example to add two identifiers to our Asset we would create a dictionary object with two items.

```python

identifiers = {"DOI": "doi:10.1038/nphys1170", "ISBN": "978-3-16-148410-0"}

```

We set the value on the option Identifier argument:


```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"
identifiers = {"DOI": "doi:10.1038/nphys1170", "ISBN": "978-3-16-148410-0"}

package = simple_asset_package("my-image.tiff",  parent_folder=folder, Identifiers=identifiers)

client.upload_zip_package(package)
```

### Descriptive Metadata

Descriptive metadata is added to the package in a similar way to the 3rd party identifiers, you create a python dictionary object and populate the dictonary key 
with the descriptive metadata schema namespace and the value of the dictionary object is a path to the xml document you would like to use.

```python

metadata = {""http://www.openarchives.org/OAI/2.0/oai_dc/" ": "./metadata/dc.xml"}

```

You can use any metadata which is a well formed XML document. The dictionary object can contain as many XML documents as you need.



```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"
metadata = {"http://www.openarchives.org/OAI/2.0/oai_dc/": "./metadata/dc.xml"}

package = simple_asset_package("my-image.tiff",  parent_folder=folder, Asset_Metadata=metadata)

client.upload_zip_package(package)
```
