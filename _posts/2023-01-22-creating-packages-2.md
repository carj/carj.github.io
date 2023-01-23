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

### Fixity

By default the `simple_asset_package()` and `complex_asset_package()` routines will create packages which contain SHA1 fixity values for each file.

You can override this default behaviour through the use of a callback which generates the required fixity on demand. The pyPreservica library provides default callbacks for SHA-1, SHA256 & SHA512

For example if you want to use SHA256 as your fixity algorithm on the preservation files use:

```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"

package = simple_asset_package("my-image.tiff",  parent_folder=folder, Preservation_files_fixity_callback=Sha256FixityCallBack())

client.upload_zip_package(package)
```

You can even choose to have different fixity algorithms for the preservaton files and the access files

```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"

package = simple_asset_package("my-image.tiff",  "my-image.jpg", parent_folder=folder, Preservation_files_fixity_callback=Sha512FixityCallBack(),  Access_files_fixity_callback=Sha256FixityCallBack())

client.upload_zip_package(package)
```

If you want to re-use existing externally generated fixity values for performance or integrity reasons then you can create a custom callback. The callback takes the filename and the path of the file which should have its fixity measured and should return a tuple containing the algorithm name and fixity value.

For example if your fixity sha256 values are stored in a spreadsheet (csv) alongside the file name you may want something similar to:


```python

class CSVFixityCallback:

    def __init__(self, csv_file):
        self.csv_file = csv_file

    def __call__(self, filename, full_path):
        with open(self.csv_file, mode='r', encoding='utf-8-sig') as csv_file:
            csv_reader = csv.DictReader(csv_file, delimiter=',')
            for row in csv_reader:
                if row['filename'] == filename
                    fixity_value = row['sha256']
                    return "SHA256", fixity_value.lower()

```

```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"

package = simple_asset_package("my-image.tiff",  parent_folder=folder, Preservation_files_fixity_callback=CSVFixityCallback("./fixity.csv"))

client.upload_zip_package(package)
```
