---
layout: post
title: Creating Packages using the pyPreservica Python SDK
---


[pyPreservica](https://pypreservica.readthedocs.io/) is a 3rd party open source (Software Development Kit) SDK for the Preservica API. 
It provides a range of services to allow users to access the full range of Preservica APIs from within simple Python scripts.
One of the most useful parts of pyPreservica is the ability to create a wide range of submission packages from 
Python scripts programmatically without requiring the user to hand create XML documents.

### Simple Packages

In pyPreservica simple packages are those which create a single Asset and which consist of a single digital object in the long term preservation representation and an optional access object.
Simple packages can be represented by the following diagram:

 ![Asset](https://pypreservica.readthedocs.io/en/latest/_images/simple_asset_package.png)
 
Simple packages contain one Content Object inside the preservation Representation and a single Content Object within an optional access Representation.
 
Simple packages can optionally contain additional descriptive metadata and 3rd party identifiers attached to the Asset.

To create a package containing a single Asset with one preservation object which will become the child of an existing folder or collection 
we can use the following Python code:

```python
package = simple_asset_package("my-image.tiff",  parent_folder=folder)
```

This will create a zipped package with the correct metadata and content ready for upload and ingest. 
The path to the newly created package is returned from the function.

To use this in a complete Python script which also uploads the package to Preservica, we pass the path to the package to an upload method. 


For example a complete runnable Python script to create and ingest the Asset would be.

```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"
package = simple_asset_package("my-image.tiff",  parent_folder=folder)
client.upload_zip_package(package)

```

This file can be saved into a text file with the extension .py and run using a Python3 run time.


Here folder is the UUID of the parent collection the Asset should be ingested into.

> **_NOTE:_**  For details on how to authenticate the pyPreservica client with your Preservica system see the section on [Authentication](https://pypreservica.readthedocs.io/en/latest/intro.html#authentication) in the pyPreservica documentation.

After ingest you should see the following in Preservica.

![Preservica Asset](/public/images/asset1.png)

If we also have an alternative version of the TIFF image such as a JPG file which we would like to be the access version, then we would use the same script, but add the access version as the second argument.

```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"
package = simple_asset_package("my-image.tiff", "my-image.jpg", parent_folder=folder)
client.upload_zip_package(package)

```
Which will give you the following Asset:

![Preservica Asset](/public/images/asset2.png)

By default `simple_asset_package()` uses the file name as the default Asset title and description, in this case the default title would be “my-image”.

![Preservica Asset](/public/images/asset3.png)

We can override that default behaviour by explicitly setting the Asset title and description by passing them as arguments to the function using optional keywords.

```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"
package = simple_asset_package("my-image.tiff", "my-image.jpg",  parent_folder=folder, Title="Asset Title", Description="Asset Description")
client.upload_zip_package(package)

```

We can also override the default “open” security tag on the Asset.


```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"
package = simple_asset_package("my-image.tiff", "my-image.jpg",  parent_folder=folder, Title="Asset Title", Description="Asset Description",  SecurityTag="closed" )
client.upload_zip_package(package)

```

Which will now populate the following fields.

![Preservica Asset](/public/images/asset4.png)


###  Multi-part Packages

If your Assets consists of multiple digital objects, for example a book with multiple pages or a multi-media object such an mp4 file and a text file containing subtitles etc. then you will need to use the function `complex_asset_package()` method.


```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"

files = ["video.mp4", "video.srt"]

package = complex_asset_package(files,  parent_folder=folder)
client.upload_zip_package(package)

```

This function works in the same way as the `simple_asset_package()` method, apart from that it accepts lists of objects, rather than a single object.
You can also pass a list of access objects, the number of preservation and access objects do not have to be equal, 
for example you can have fewer access objects in the Asset.


```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"

pres_files = ["page1.tif", "page2.tif", "page3.tif"]
access_files = ["book.pdf"]

package = complex_asset_package(pres_files, access_files, parent_folder=folder)
client.upload_zip_package(package)

```

`complex_asset_package()` will always retain the order of the objects in the lists, this way you can preserve the correct ordering of objects which is useful when rendering the Asset. 


![Preservica Asset](/public/images/asset5.png)


###  Multiple Representations

If you need more than one preservation or access representation then the method to use is `generic_asset_package()`, this provides the most flexibility when creating submissions at the price of slightly more complexity.

This function allows arbitrary numbers of both preservation and access representations to be created.
Instead of a list of objects you now pass a dictionary object, the key is the representation name and the value is the list of files.


```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"

preservation_rep = dict()
preservation_rep["Master"] = ["page1.tif", "page2.tif"," page3.tif"]
preservation_rep["Black White"] = ["page1.tif", "page2.tif"," page3.tif"]
preservation_rep["Greyscale"] = ["page1.tiff", "page2.tiff"," page3.tiff"]

access_rep = dict()
access_rep["Multi-Page Access"] = ["page-1.jpg", "page-2.jpg"," page-3.jpg"]
access_rep["Single-Page Access"] = ["book.pdf"]

package = generic_asset_package(preservation_rep, access_rep, parent_folder=folder)


```

The additional keyword arguments used by `simple_asset_package()` and `complex_asset_package()` such as Title, Description etc are still available.
Preservica will render the first access representation, so the viewer you want to use needs to be the first entry in the access dictionary. 

Using the example above, if you want to use the multi-page book viewer as the default renderer, make “Multi-Page Access” the first entry, 
if you want the PDF viewer to be the default renderer, then make “Single-Page Access” the first dictionary entry.

![Preservica Asset](/public/images/asset6.png)

