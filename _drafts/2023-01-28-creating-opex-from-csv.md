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
<oai_dc:dc  xmlns:oai_dc="http://www.openarchives.org/OAI/2.0/oai_dc/">
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


```

The root element name is ```dc``` and the root element namespace is ```http://www.openarchives.org/OAI/2.0/oai_dc/```

By default all the element names are within the root element namespace. 

![Spreadsheet Converter](/public/images/converter8.PNG)

In this case the root element namespace and the Dublin Core elements should be in different namespaces, (15 term Dublin Core elements actually live inside the "http://purl.org/dc/elements/1.1/" namespace) so this is a good example of where we add a prefix to the column names, this allows us to associate additional namespaces to the elements.

![Spreadsheet Converter](/public/images/converter9.PNG)


### Formatting the Output

After the CSV has been uploaded you need to select which column should be used to name the XML documents. This should be a column containing either the filename or other unique information 

![Spreadsheet Converter](/public/images/converter10.PNG)

###  Naming Convention

![Spreadsheet Converter](/public/images/converter11.PNG)

This is where you can decide on what type of metadata you would like to export.

The first two options (.xml and .metatadata) create simple XML documents which would look like:

```xml

<?xml version="1.0" encoding="UTF-8"?>
<dc xmlns="http://www.openarchives.org/OAI/2.0/oai_dc/" xmlns:dc="http://purl.org/dc/elements/1.1/">
	<filename>image-0001.jpg</filename>
	<dc:title>Title 001</dc:title>
	<dc:creator>James Carr</dc:creator>
	<dc:subject>Sheffield</dc:subject>
	<dc:description/>
	<dc:publisher/>
	<dc:contributor/>
	<dc:date/>
	<dc:type/>
	<dc:format/>
	<dc:identifier/>
	<dc:source/>
	<dc:language/>
	<dc:rights/>
</dc>


````

The only difference being the file extension. The 3rd option wraps the XML in a OPEX header element ready for upload through the PUT tool.

```xml

<?xml version="1.0" encoding="UTF-8"?>
<opex:OPEXMetadata xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:oai_dc="http://www.openarchives.org/OAI/2.0/oai_dc/" xmlns:opex="http://www.openpreservationexchange.org/opex/v1.2">
	<opex:DescriptiveMetadata>
		<oai_dc:dc>
			<dc:title>Title 001</dc:title>
			<dc:creator>James Carr</dc:creator>
			<dc:subject>Sheffield</dc:subject>
			<dc:description/>
			<dc:publisher/>
			<dc:contributor/>
			<dc:date/>
			<dc:type/>
			<dc:format/>
			<dc:identifier/>
			<dc:source/>
			<dc:language/>
			<dc:rights/>
		</oai_dc:dc>
	</opex:DescriptiveMetadata>
</opex:OPEXMetadata>

```


