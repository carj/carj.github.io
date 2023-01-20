---
layout: post
title: Creating Preservica Packages using the Python SDK
---


[pyPreservica](https://pypreservica.readthedocs.io/) is a 3rd party open source (Software Development Kit) SDK for the Preservica API. 
It provides a range of services to allow users to access the full range of Preservica APIs from within simple Python scripts.
One of the most useful parts of pyPreservica is the ability to create a wide range of submission packages from 
Python scripts programmatically without requiring the user to hand create XML documents.

### Simple Packages

In pyPreservica simple packages are those which create a single Asset and which consist of a single digital object in the long term preservation representation and an optional access object.
Simple packages can be represented by the following diagram:

 ![Asset](https://pypreservica.readthedocs.io/en/latest/_images/simple_asset_package.png)
 
Simple packages can optionally contain additional descriptive metadata and 3rd party identifiers.
To create a package containing a single Asset with one preservation object which will be a child of a specified folder we can use the following Python code:

```python
package = simple_asset_package("my-image.tiff",  parent_folder=folder)
```

This will create a zipped package with the correct metadata and content ready for upload and ingest. The path to the newly created pacakge is returned from the function.

To use this in a complete Python script which also uploads the package to Preservica we pass the path to the package to an upload method. 
For example we would have:
