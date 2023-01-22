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
