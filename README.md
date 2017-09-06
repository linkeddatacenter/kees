KEES: Knowledge Exchange Engine Schema 
=======================================

**WARNING: WORKING IN PROGRESS**

In order to let computers to work for us, they must understand data. 
Not only grammar and syntax (like in EDI) but the real meaning of things. 
When this is clear, machines can recognize, manage data and make decisions even when their provenance and 
quality is not totally known and data are incomplete.
The same concerns apply to the knowledge itself: to share knowledge we need a language that is understandable both for humans and machines.

This is the source code repository for the KEES language profile that is used to describe a knowledge configurations.
Semantic web agents and humans can use shuch descriptions to populate, merge and share a domain specific knowledge bases. 

See [KEES project presentation](https://docs.google.com/presentation/d/1mv9XO0Q9QFxSphWzT_68Q4aXd9sgqWoY7njomH8eaPQ/pub?start=false&loop=false&delayms=5000)


## Introduction

This document describes vocabularies and constraints to be used to describe Knowlege Base.

## Terminology used in the KEES language profile

An **Language Profile** is a specification that re-uses terms from one or more base
standards, adding more specificity by identifying mandatory, recommended and
optional elements to be used for a particular application, as well as recommendations
for controlled vocabularies to be used.

A **Dataset** is a collection of data, published or curated by a single source, and
available for access or download in one or more formats.

A **Trust map** is a rank about the quality of a dataset.

A **Named graph** is nn RDF dataset as defined in [RDF specifications](http://www.w3.org/TR/rdf11-primer/)

A **Knowledge Base** is a semantic system where information is described as a set of statements according with the W3C standard 
Resource Description Framework (RDF). A  Knowledge Base is composed by two disjoint set of statements: *TBox* and *ABox*. 

**TBox statements** describe a system in terms of controlled vocabularies, rules and axioms. TBox statements sometimes associate with object-oriented classes.
TBox statements tend to be more permanent within a knowledge base and are often grouped in "ontologies" that describe a specific 
knowledge domain (e.g. business entities, people, goods, friendship, offering, geocoding, etc, etc).

**ABox statements** associate with instances of classes defined by TBox statements. ABox statements are much more dynamic in nature and 
are populated from datasets available in the web or by reasonings.

A **Rule** describes how to generate/validate knowledge statemensts using an algoritmic approach

An **Axiom** describes how to generate/validate RDF statemensts using a declarative approach

**Reasoning** is the process of inferring new rules from existing data.

**Graph database**: a database that implements a Knowledge Base  as a set of ABox graphs and ABox graphs

**TBox graph** is a named graph tah contains only TBox statements as RDF triples.

**ABox graph** is a named graph tah contains only ABox statements as RDF triples.

**Configuration graph** is an ABoxGraph that contains statements that describe a knowledge base as RDF triples.

**Inferred data graph** is an ABoxGraph that contains statements derived from a reasoning as RDF triples.

**Linked Data data graph** is an ABoxGraph that contains statements derived from dastasets (i.e. facts).

A **KEES agent** is a semantic web agent that knows KEES language profile and is able to do actions on a 
knowledge base taking into account the statements contained in all configuration graphs.

A special type of ABox statements are those that are inferred through an automatic reasoning process, 
that is the evaluation of rules and/or axioms explicitly asserted or derived from TBox.

To describe a knowledge base, the KEES Language Profile reuses terms from various existing specifications. Classes and
properties specified in the next sections have been taken from the following
namespaces:

- adms: http://www.w3.org/ns/adms#
- dcat: http://www.w3.org/ns/dcat#
- dct: http://purl.org/dc/terms/
- dcmitype: http://purl.org/dc/dcmitype/
- foaf: http://xmlns.com/foaf/0.1/
- owl: http://www.w3.org/2002/07/owl#
- rdfs: http://www.w3.org/2000/01/rdf-schema#
- schema: http://schema.org/
- skos: http://www.w3.org/2004/02/skos/core#
- spdx: http://spdx.org/rdf/terms#
- xsd: http://www.w3.org/2001/XMLSchema#
- qb: http://purl.org/linked-data/cube#
- daq: http://purl.org/eis/vocab/daq# 
- prov: http://www.w3.org/ns/prov#
- void: http://rdfs.org/ns/void#
- sd: http://www.w3.org/ns/sparql-service-description#
- kees: http://linkeddata.center/kees/v1#

## KEES by examples

### simple web resource (re)loading

This states that an ABoxGraph SHOULD exist in the knowledge base and that graph should be loaded with the content of the web 
resource "http://data.example.com/dataset1.ttl"

```
[] a kees:LinkedDataGraph; dct:source <http://data.example.com/dataset1.ttl> .
```

These two RDF triples are equivalent to:

```
[] a kees:LinkedDataGraph,kees:ABoxGraph, sd:namedGraph, dcmitype:Collection;
	sd:name  <http://data.example.com/dataset1.ttl> ;
	dct:source <http://data.example.com/dataset1.ttl> ;
	dct:accrualPolicy kees:on_source_change ;
	dct:accrualMethod kees:load_resource;
.

[]	dcat:Distribution, void:Dataset ;
	dcat:accessURL <http://data.example.com/dataset1.ttl>
	void:dataDump <http://data.example.com/dataset1.ttl>
.

[] a qb:Observation ;
	daq:computedOn [ a sd:NamedGraph; sd:name <http://data.example.com/dataset1.ttl> ] ; 
    daq:metric kees:trustRank;
    daq:value "0.5"^^xsd:double;
	daq:isEstimated true .
.

```

A KEES compliant agent SHOULD check if <http://data.example.com/dataset1.ttl> resource is newer than  <http://data.example.com/dataset1.ttl> named graph in the knowledge base; if yes it COULD execute the following sparql update statements:

```
DROP SILENT GRAPH <http://data.example.com/dataset1.ttl> ;
LOAD <http://data.example.com/dataset1.ttl> INTO GRAPH <http://data.example.com/dataset1.ttl> ;
INSERT DATA {
	GRAPH <http://data.example.com/dataset1.ttl> {
		[] a prov:Entity, sd:NamedGraph ; 
			sd:name <http://data.example.com/dataset1.ttl> ;
			dct:modified "here current date"^xsd:date ;
			prov:wasGeneratedBy [ 
				a prov:Activity , kees:sparql_upload_activity ;
				prov:wasAssociatedWith kees:load_resource_processor ;
				prov:used <http://data.example.com/dataset1.ttl>
			] ;
		.
		# following properties COULD be inferred from http "Last-Modified:", "ETAG" and content type headers
		<http://data.example.com/dataset1.ttl> a prov:Entity , foaf:Document ;
			dct:modified  "2017-03-30"^^xsd:date ;
			dct:identifier "123456789" ;
		    dct:format "text/turtle" ;
		.	
	}
}
```


### simple protected dataset (re)loading
This states that an ABoxGraph named *http://data.example.com/dataset1* SHOULD exist in the knowledge base and that graph should be loaded with the content of the web 
resources *http://data.example.com/dataset1.ttl?page=1* and *http://data.example.com/dataset1.ttl?page=2* that are protected with a basic http autentication.

```
[] a kees:LinkedDataGraph; sd:name <http://data.example.com/graph/dataset> dct:source <http://data.example.com/dataset1.rdf> -
[]  dcat:accessURL <http://data.example.com/dataset1.rdf> ;
	void:dataDump 
	    <http://data.example.com/dataset1.ttl?page=1>,
        <http://data.example.com/dataset1.ttl?page=2>
.
[] a kees:BasicHttpAuthentication ;
	kees:realm "example.com private area";
	kees:uriRegexPattern "^http://data.example.com/dataset1.ttl"
.
```

A KEES compliant agent SHOULD get someway the username and password required by the http basic authentication; it could use rdfs:label and rdfs:comment to ask these data to user.
Than, using such credentials, check if <http://data.example.com/dataset1.rdf> resource is newer than the creation date of the
<http://data.example.com/graph/dataset> named graph in the knowledge base.
If yes it SHOULD load the two datadump resources in a cache and loads them in the same graph in kb.

### simple web resource incremental loading

This states that a n ABoxGraph SHOULD exists in the knowledge base and that graph should updated appending the content of the web 
resource "http://data.example.com/dataset1.ttl"

TODO:

### simple web resource with custom accrual policy

TODO:


### simple web resource with custom accrual method

TODO:

## A complete example

TODO:



## Contributing to the site

A great way to contribute to the site is to create an [issue](https://github.com/linkeddatacenter/kees/issues) on GitHub when you encounter a problem or something. We always appreciate it. You can also edit the code by yourself and create a pull request.


All stuff here in the Creative Common (unless otherwise noted)
