---
layout: post
title: Creating OPEX XML from Spreadsheets
---

This is a brief user guide for creating OPEX XML documents using the 3rd party [Spreadsheet converter](https://pypreservica.pythonanywhere.com/).

![Spreadsheet Converter](/public/images/converter1.png)


This website allows you to upload a spreadsheet containing descriptive metadata and generate XML documents compatible with Preservica.

### Background

The spreadsheet converter makes a few simple assumptions about how data is stored in the spreadsheet. It assumes that each row of the spreadsheet contains descriptive metadata for a single Preservica Asset. For example one row corresponds to the metadata for a single document or image etc.

The spreadsheet should contain a header row, the column names in the header will become the metadata attributes within the exported XML documents.

For example, for Dublin Core metadata you may have a spreadsheet similar to:

![Spreadsheet Converter](/public/images/converter3.png)


