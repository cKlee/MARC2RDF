# easyM2R Deprecated

**This repository is deprecated in favour of the more powerful tool [Catmandu](http://librecat.org/)**

easyM2R is a php-based attempt to easily convert MARC data to RDF.

It's easy because you only need

  * your MARC data as a file or a string
  * a valid JSON-LD file which shows how your data should look like in RDF
  * PHP installed in version 5.3.x or higher

## Change notes

* **0.2beta**: easyM2R is now more easily to use. It now uses [MarcSpec](https://github.com/cKlee/php-marc-spec) a MARC spec as string parser and validator, which makes a lot of default callback functions deprecated. For custom callback functions the referenced data of the MARC data is always provided within the variable $_params. Thus you may not have to access the MARC record via File_MARC.  

# Credits

easyM2R is build upon the following software

* [File_MARC](https://github.com/pear/File_MARC) by Dan Scott, Copyright (c) 1991, 1999 Free Software Foundation, Inc.
* [JsonLD](https://github.com/lanthaler/JsonLD) processor for PHP and [IRI](https://github.com/lanthaler/IRI) Copyright (c) by Markus Lanthaler
* [EasyRdf](https://github.com/njh/easyrdf) Copyright (c) by Nicholas J Humfrey
* [forceutf8](https://github.com/neitanod/forceutf8) by Sebastián Grignoli
* [MarcSpec](https://github.com/cKlee/php-marc-spec)

# Overview

``` {.ditaa}
    +--------------+
    | MARC data    +----+
    | in a file    |    |
    +--------------+    |   +--------------+
                        +-->| MARCFILE2RDF +-----+
    +--------------+    |   +--------------+     |
    | MARCXML data +----+                        |
    | in a file    |                             |
    +--------------+                             |   +-------------+
                                                 +-->| data as RDF |
    +--------------+                             |   +-------------+
    | MARC data    +----+                        |
    | as a string  |    |                        |
    +--------------+    |   +----------------+   |
                        +-->| MARCSTRING2RDF +---+
    +--------------+    |   +----------------+
    | MARCXML data +----+
    | as a string  |
    +--------------+
```

# Installation

The best way to download easyM2R is to clone the repository recursively.

```
git clone --recursive https://github.com/cKlee/easyM2R.git
```

You can also download the [releases](https://github.com/cKlee/easyM2R/releases) and unpack them at the desired place.

# Quickstart using the command line

Navigate to the easyM2R base directory where you'll find the file 'tordf.php'. At the command line type

```
php tordf.php -s <PATH_TO_YOUR_MARC_SOURCE> -i <MARC_INPUT_FORMAT> -o <RDF_OUTPUT_SERIALIZATION>
```

This will output your MARC data in RDF with the desired output serialization. See [Using the command line interface] for further explanation of the command line options.

# Using the command line interface

[Using the command line interface]: #using-the-command-line-interface

With the easyM2R command line interface you can only use MARC data from a file.

The command line interface is called via the script **tordf.php** with the command **php tordf.php**. At the command line interface you have these options

* -t Path to the jsonld template
* -o The output format must be one of 'jsonld', 'json', 'php', 'ntriples', 'turtle', 'rdfxml', 'dot', 'n3', 'png', 'gif', 'svg'
* -s The Path to your MARC source file
* -i The MARC input format. 'xml' for MARCXML source
* -c The path to your custom callback functions directory
* -b The base IRI for each MARC record in RDF
* -p 0 or false for merging graphs before output (default might take a lot of memory) or 1 or true to output each single record graph

All options are optional. But if you want to convert your own MARC data, you have to set the -s option at least.

## Testing easyM2R via command line

You can test easyM2R by using the example marc data with the command

```
php tordf.php -s examples/marc/e-discover.mrc
```

For image formats you might pipe the output to a file

```
php tordf.php -s examples/marc/e-discover.mrc -o png > e-discover.png
```

# Using easyM2R in a custom PHP script

Using easyM2R within a custom PHP script is necessary if you fetch the MARC data from a stream as a string and pass on to easyM2R. This sample code gives a short insight how a custom script could look like:

```php
<?php
// use namespace
use CK\MARC2RDF as m2r;

// always include the autoload.php
include('path/to/easyM2R/autoload.php');

// include your custom callback scripts
foreach(glob('my_callback/callback_*.php') as $filename) include $filename;

// fetch your MARC data here
$xml_string = fetch your...

// initiate easyM2R
$easyM2R = new m2r\MARCSTRING2RDF('../template/default.jsonld',$xml_string,'xml');

// print pretty RDF for browser
print '<pre>'.htmlspecialchars($easyM2R->output('turtle')).'</pre>';
```

## Choosing the right class

easyM2R provides to main classes **MARCFILE2RDF** and **MARCSTRING2RDF**. Use the MARCFILE2RDF class if your data resides in a file and use MARCSTRING2RDF if you want to pass your MARC data as a string to easyM2R.

### Class CK\MARC2RDF\MARCFILE2RDF

The MARCFILE2RDF class accepts 4 parameters:

* @param string The local path or URL of the jsonld template file
* @param string Path to MARC data as a file
* @param null|string The MARC format must be one of 'jsonld', 'json', 'php', 'ntriples', 'turtle', 'rdfxml', 'dot', 'n3', 'png', 'gif', 'svg'
* @param null|string The base IRI for each MARC record in RDF
* @param bool $perRecord Do not merge, just make the current recordGraph available

The MARCFILE2RDF class has 3 public methods:

* CK\\MARC2RDF\\MARCFILE2RDF::\__construct($jsonld_file,$marc_file,$marc_format = null,$base = null,$perRecord = false)
* CK\\MARC2RDF\\MARCFILE2RDF::next()
* CK\\MARC2RDF\\MARCFILE2RDF::output($format = 'jsonld',jld\GraphInterface $graph = null)


### Class CK\MARC2RDF\MARCFSTRING2RDF

The MARCSTRING2RDF class accepts 4 parameters:

* @param string The local path or URL of the jsonld template file
* @param string MARC data as string
* @param null|string The MARC format must be one of 'jsonld', 'json', 'php', 'ntriples', 'turtle', 'rdfxml', 'dot', 'n3', 'png', 'gif', 'svg'
* @param null|string The base IRI for each MARC record in RDF
* @param bool $perRecord Do not merge, just make the current recordGraph available

The MARCSTRING2RDF class has 3 public methods:

* CK\\MARC2RDF\\MARCSTRING2RDF::\__construct($jsonld_file,string $marc_string,$marc_format = null,$base = null,$perRecord = false)
* CK\\MARC2RDF\\MARCSTRING2RDF::next()
* CK\\MARC2RDF\\MARCSTRING2RDF::output($format = 'jsonld',jld\GraphInterface $graph = null)

### Public Properties

```php
/**
* @var string The local path or URL of the jsonld template file 
*/
public $jsonld_file;

/**
* @var string The local path or URL of the MARC source
*/
public $marc_source;

/**
* @var string the base IRI for the resource
*/
public $base;

/**
* @var GraphInterface Merge of all record graphs 
*/
public $newGraph;

/**
* @var GraphInterface The currently created graph
*/
public $recordGraph;

/**
* @var string|null The name of the template graph, which should be used
*/
public $graph_name = null;

/**
* @var string The desired output format
*/
public $format = 'jsonld';
```

### XML caveat

While it is possible to transform MARCXML data, there is a problem with namespaces within the XML. FILE_MARC does not handle namespaces until now. This behaviour may change in the future.

# Configuration

The configuration is done via a JSON-LD document called template. The template is the blue print for every graph resulting from a MARC record. The template has to follow some ground rules, but whatever you do in the template, remember that it must be a valid JSON-LD file. You can test the validity of your template via the [JSON-LD Playground](http://json-ld.org/playground/).

As an example a default template is provided [**`default.jsonld`**](template/default.jsonld).

## MARC spec

In the template you want to access the MARC fields and subfields. This is done via a **MARC spec**. A MARC spec has a defined syntax, which is described under http://cklee.github.io/marc-spec/marc-spec.html .

In short, a MARC spec  

```
fieldTag (["~" characterPositionOrRange] / [subfieldTags] ["_" indicators])
```

That is, if you want to access subfield 'a' in field '245', the MARC spec is '245a'. For MARC control fields there are no subfields but chracter position or range prefixed with the character "~". The MARC spec 008~0-4 is a reference to the characters 1 (first character with index 0) to 5 (fifth charcter with index 4) of MARC field 008. Character position and range can also be applied to the Leader. The MARC record Leader in a MARC spec is defined through 'LDR'. The MARC spec LDR~0 is a reference to the first character of the Leader.

A MARC spec is only recognized, if it is prefixed with the easyM2R namespace (see [@context]).

If there are multiple subfields with the same name in one field, there also will be created multiple nodes.

There is also a more powerful way to access MARC fields via [callbacks].

## @context

[@context]: #context

In the template you must create a **@context** node. In the @context node the only mandatory entry is the easyM2R namespace declaration. 

```json
{
    "@context": {
        "marc2rdf": "http://my.arbitratynamespace.com#"
    }
}
```

The prefix of the easyM2R namespace must be 'marc2rdf'. The namespaces identifier is also the base IRI to your RDF resources. Choose a custom identifier, which must end with '/' or '#'. You can overwrite the base IRI for your RDF resources by setting the $base param for the MARCFILE2RDF or MARCSTRING2RDF class or with the -b option for the command line tool.

## @graph

The **@graph** is the template for each MARC record.

Within the @graph you must define the resources **@id**. The value of the @id consists of the easyM2R namespace prefix and a MARC spec. I.e.

```json
{
    "@id": "marc2rdf:001"
}
```

In this example, if the data in the control field '001' is '123245', then your resources IRI will be 'http://my.arbitratynamespace.com#12345'.

Now define your properties and objects. Regardless of the node type you create (resource, typed value or untyped value) if you want to access a MARC field/subfield always use the easyM2R namespace as a prefix. Otherwise the MARC spec will not be recognized as one.

## callbacks

MARC data is not always that easy to access. Sometimes you have to check the context, or look up substrings. Or if you want to join data from subfields or shape data in a different way, there is a powerful way to do this via **callbacks**.

Callbacks are functions that are called if you specify them in the template. There are some default callbacks (see [default callbacks]) but you can write your own custom callbacks very easily (see [create custom callbacks]).

In the template if you want to call a callback, prefix the callback name with the easyM2R namespace prefix 'marc2rdf'. This could look like this example

```json
"oclcnum":{"@value": "marc2rdf:callback_prefix_in_parentheses(035a,OCoLC)"}
```

See [default callbacks] for specific usage.

If you want to use a callback function to return a value for the rdf:type property, then you can't use the JSON-LD syntax token '@type'. The solution is to define the property 'type' within the @context node and use that instead of '@type'.

### default callbacks

[default callbacks]: #default

There are some predefined default callback functions that are listed here. Each default callback function takes one to n parameters (often the number is fixed), which are either MARC specs or nonspecs. Nonspecs must always be urlencoded.

#### callback_template

* param 1: MARC spec
* param 2: regex replace pattern

Return data in the shape of the param m. Data of first param is filled in '$0', second in '$1' and so on...

Example

```json
marc2rdf:callback_template(260abc,$0%20%3A%20$1%2C%20$2)
```

(since $0 : $1, $2 is url encoded $0%20%3A%20$1%2C%20$2)

leads to something like

```
Detmold : Kreis Lippe, Der Landrat
```

#### callback\_substring_after

* param 1: MARC spec
* param 2: substring

Return data comes after substring.

#### callback\_subfield_context

* param 1: MARC spec
* param 2: MARC spec
* param 3: context

Returns data from param 1, if context is substring of data from param 2. 

#### callback\_string_contains

* param 1: MARC spec
* param 2: containing string

Return data if data from param 1 contains string in param 2.

#### callback\_prefix_in_parentheses

* param 1: MARC spec
* param 2: prefixed string

Return data without prefix from param 1 if it is prefixed with param 2 in parentheses.

#### callback\_make_iri

* param 1: MARC spec
* param 2: IRI prefix

Return IRI consisting of value of param 2 and data from param 1. 

#### callback_join

* param 1: MARC spec
* param 2: join character

Return data from param 1 joined with character in param 2. 

#### callback\_make\_bn

* param 1: MARC spec

For each data element returned a whole new blank node is created having all properties of the original defined blank node.

### create custom callbacks

[create custom callbacks]: #custom

Custom callback functions names must start with 'callback', otherwise they cannot be called.

A callback functions takes two parameters. The first is the MARC record and the second is an array containing MARC specs, nonspes and the resulting data.

The first line of your custom callback function might look like

    function callback_mycustom(File_MARC_Record $record, array $_params)

The variable $record is a File_MARC_Record object. This you can access MARC data via its methods (see http://pear.php.net/package/File_MARC/docs for documentation). Although you can access the MARC record data via File_MARC there is also the possibility to process the provided data within the index 'data' of the variable $_param.

For an example the statement in the JSON-LD template

```json
"oclcnum":{"@value": "marc2rdf:callback_prefix_in_parentheses(035a_01,OCoLC)"}
```

the variable $_params in called default callback function callback\_prefix\_in\_parentheses is an associative array that might look like:

```php
[specs] => Array
        (
            [0] => Array
                (
                    [field] => 035
                    [subfield] => Array
                        (
                            [a] => a
                        )
                    [indicator1] => 0
                    [indicator2] => 1
                )
        )

[nonspecs] => Array
    (
        [0] => OCoLC
    )

[rootId] => _:b0

[data] => Array
    (
        [0] => (OCoLC)723997824
    )
```

See usage of key 'rootId' under [dealing with dynamic blank nodes].

Return the data at the end of the function. Then include your custom callbacks in your script like

```php
foreach(glob('my_callback/path/callback_*.php') as $filename) include $filename;
```

or use the -c option for the command line interface.

### dealing with dynamic blank nodes

[dealing with dynamic blank nodes]: #bnodes

For example you specified a blank node in your template

```json
"@id": "marc2rdf:001_0",
"property1":
{
    "@id": "_:bnode_1",
    "@type": "Sometype",
    "property2": {"@value": "marc2rdf:866z"}
    "property3": {"@value": "marc2rdf:866y"}
}
```

and you want for each data of subfield 'z' and 'y' in field '866' to create a blank node, in your callback function the returning array might look like this

```php
// subfield z
[_:b0] => value 1
[_:b1] => value 2
[_:b2] => value 3

// subfield y
[_:b0] => value 4
[_:b1] => value 5
[_:b2] => value 6
```

This would result in something like

```rdf
<http://my.arbitratynamespace.com#12345>
    someprefix:property1 [
        a Sometype ;
        someprefix:property2 "value 1" ;
        someprefix:property2 "value 4"
    ], [
        a Sometype ;
        someprefix:property2 "value 2" ;
        someprefix:property2 "value 5"
    ], [
        a Sometype ;
        someprefix:property2 "value 3" ;
        someprefix:property2 "value 6"
    ].
```

But how do you know what blank node identifiers to use? This is the point where you'll need the value of the key 'rootId' in the var $_params. This value is the id of the currently created node. Just make sure that the first key in your returning array is this id and that all other keys are with a higher count.

# Graph clean up

The resulting graph will be cleaned of *empty* blank nodes only having a rdf:type property.
