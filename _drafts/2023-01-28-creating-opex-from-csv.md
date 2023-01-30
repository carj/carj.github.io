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

The metadata is added to the rows under the column headings 

![Spreadsheet Converter](/public/images/converter5.PNG)

and the spreadsheet should be exported to UTF-8 CSV.

![Spreadsheet Converter](/public/images/converter6.PNG)

You are now ready to upload the metadata.

### Namespaces

The first field to enter on the website askes for a *Root Element* name

![Spreadsheet Converter](/public/images/converter7.PNG)

This field controls the name of the XML metadata root element. This is the element that all the spreadsheet column names are children of.

For example, to create OAI-DC type metadata such as:


```xml

<?xml version="1.0" encoding="UTF-8"?>
<opex:OPEXMetadata xmlns:oai_dc="http://www.openarchives.org/OAI/2.0/oai_dc/" xmlns:opex="http://www.openpreservationexchange.org/opex/v1.2">
	<opex:DescriptiveMetadata>
		<oai_dc:dc>
			<oai_dc:title>Title 001</oai_dc:title>
			<oai_dc:creator>James Carr</oai_dc:creator>
			<oai_dc:subject>Sheffield</oai_dc:subject>
			<oai_dc:description/>
			<oai_dc:publisher/>
			<oai_dc:contributor/>
			<oai_dc:date/>
			<oai_dc:type/>
			<oai_dc:format/>
			<oai_dc:identifier/>
			<oai_dc:source/>
			<oai_dc:language/>
			<oai_dc:rights/>
		</oai_dc:dc>
	</opex:DescriptiveMetadata>
</opex:OPEXMetadata>

```

The root element name is ```dc``` and the root element namespace is ```http://www.openarchives.org/OAI/2.0/oai_dc/```

![Spreadsheet Converter](/public/images/converter8.PNG)

In this case the root element namespace and the Dublin Core elements should be in different namespaces, (15 term Dublin Core elements actually live inside the "http://purl.org/dc/elements/1.1/" namespace) so this is where we add a prefix to the column names, this allows us to associate additional namespaces to the prefixes.

![Spreadsheet Converter](/public/images/converter9.PNG)


### Formatting the Output

After the CSV has been uploaded you need to select which column should be used to name the XML documents. This should be a column containing either the filename or other unique information 

![Spreadsheet Converter](/public/images/converter10.PNG)


