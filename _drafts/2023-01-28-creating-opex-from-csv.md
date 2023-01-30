---
layout: post
title: Creating OPEX XML from Spreadsheets
---

This is a brief user guide for creating OPEX XML documents using the 3rd party [Spreadsheet Converter](https://pypreservica.pythonanywhere.com/).

![Spreadsheet Converter](/public/images/converter1.PNG)


This website allows you to upload a spreadsheet containing descriptive metadata and generate XML documents compatible with Preservica.

### Background

The spreadsheet converter makes a few simple assumptions about how data is stored in the spreadsheet. It assumes that each row of the spreadsheet contains descriptive metadata for a single Preservica Asset. For example one row corresponds to the metadata for a single document or image etc.

Your spreadsheet should contain a header row, the column names in the header will become the metadata attributes within the exported XML documents.

For example, for Dublin Core metadata you may have a spreadsheet similar to:

![Spreadsheet Converter](/public/images/converter3.PNG)

You may also have a column which contains the name of the digital object to which the metadata refers such as filename etc. The name of this column is not important. If you dont have a special column such as filename etc then one of the other column names must contain some unique information.

![Spreadsheet Converter](/public/images/converter4.PNG)

The spreadsheet column names can also contain prefixes, these can be used to add additional XML namespaces into the XML documents. The prefixes are seperated by ":" from XML attribute names.

Prefixes can be useful for XML schema's such as Dublin Core where the XML attributes live inside a different namespace to the main XML root element.

![Spreadsheet Converter](/public/images/converter2.PNG)
