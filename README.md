KEES: Knowledge Exchange Engine Schema 
=======================================

*** WARNING THIS SPECIFICATION IS STILL ON WORKING ***

In order to let computers to work for us, they must understand data. 
Not only grammar and syntax (like in EDI) but the real meaning of things. 
When this is clear, machines can recognize, manage data and make decisions even when their provenance and 
quality is not totally known and data are incomplete. 
To enable this, we need to share knowledge using a language that is understandable both for humans and machines.

This is the source code repository for the KEES language profile that is used to describe a knowledge base configurations.
Semantic web agents and umans can use this description to populate,, merge and share a domain specific knowledge. 

See [KEES project presentation](https://docs.google.com/presentation/d/1mv9XO0Q9QFxSphWzT_68Q4aXd9sgqWoY7njomH8eaPQ/pub?start=false&loop=false&delayms=5000)


## Introduction

This document describes vocabularies and constraints to be used to describe Knowlege Base configurations.

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
Knowledge Base is implemented as a set of named graph

A **Rule** describes how to generate/validate RDF statemensts using an algoritmic approach

An **Axiom** describes how to generate/validate RDF statemensts using a declarative approach

**Reasoning** is the process of inferring new RDF statements from existing RDF statements using rules and axioms.

**TBox statements** describe a system in terms of controlled vocabularies, rules and axioms. TBox statements sometimes associate with object-oriented classes.
TBox statements tend to be more permanent within a knowledge base and are often grouped in "ontologies" that describe a specific 
knowledge domain (e.g. business entities, people, goods, friendship, offering, geocoding, etc, etc).

**ABox statements** associate with instances of classes defined by TBox statements. ABox statements are much more dynamic in nature and 
are populated from datasets available in the web or by reasonings.

**TBox graph** is a named graph tah contains only TBox statements.

**ABox graph** is a named graph tah contains only ABox statements.

**Configuration graph** is an ABoxGraph that contains statements that describe a knowledge base.

**Inferred data graph** is an ABoxGraph that contains statements derived from a reasoning

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
- foaf: http://xmlns.com/foaf/0.1/
- owl: http://www.w3.org/2002/07/owl#
- rdfs: http://www.w3.org/2000/01/rdf-schema#
- schema: http://schema.org/
- skos: http://www.w3.org/2004/02/skos/core#
- spdx: http://spdx.org/rdf/terms#
- xsd: http://www.w3.org/2001/XMLSchema#
- vcard: http://www.w3.org/2006/vcard/ns#
- qb: http://purl.org/linked-data/cube#
- daq: http://purl.org/eis/vocab/daq# 
- prov: http://www.w3.org/ns/prov#
- void: http://rdfs.org/ns/void#
- sd: http://www.w3.org/ns/sparql-service-description#
- kees: http://linkeddata.center/kees/v1#

## KEES by examples

### simple web resource (re)loading

This states that a n ABoxGraph SHOULD exists in the knowledge base and that graph should be loaded with the content of the web 
resource "http://data.example.com/dataset1.ttl"

```
[] a kees:ABoxGraph; dct:source <http://data.example.com/dataset1.ttl> .
```

These two RDF triples MUST be equivalent to:

```
[] a kees:ABoxGraph, sd:namedGraph;
	sd:name  <http://data.example.com/dataset1.ttl> ;
	dct:source <http://data.example.com/dataset1.ttl> ;
	dct:accrualPolicy [	
		a kees:UpdateGraphPolicy;
		dct:accrualMethod [ a kees:SparqlUpload ] ;
		kees:onFetchingError [ a kees:RetentionPolicy ; kees:hasResilience 0 ] ;
	]
.

[]	void:Dataset ;
	void:dataDump <http://data.example.com/dataset1.ttl>;
.

[] a qb:Observation ;
	daq:computedOn [ a sd:NamedGraph; sd:name <http://data.example.com/dataset1.ttl> ] ; 
    daq:metric kees:trustRank;
    daq:value "0.5"^^xsd:double;
	daq:isEstimated true .
.

```

For example a KEES compliant agent SHOULD check if <http://data.example.com/dataset1.ttl> resource is newer than 
 <http://data.example.com/dataset1.ttl> named graph in the knowledge base; if yes it COULD execute the following sparql update statements:

```
DROP SILENT GRAPH <http://data.example.com/dataset1.ttl> ;
LOAD <http://data.example.com/dataset1.ttl> INTO GRAPH <http://data.example.com/dataset1.ttl> ;
INSERT DATA {
	GRAPH <http://data.example.com/dataset1.ttl> {
		[] a prov:Entity, sd:NamedGraph ; 
			sd:name <http://data.example.com/dataset1.ttl> ;
			dct:modified "here current date"^xsd:date ;
			prov:wasGeneratedBy [ 
				a prov:Activity , kees:SparqlUploadActivity ;
				prov:wasAssociatedWith [ a kees:Agent ] ;
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

Note that the property `kees:supportsLDPPP false` means that the support of Linked Data Platform Paging Protocol MUST be disabled.

### simple protected web resource (re)loading

This states that an ABoxGraph SHOULD exists in the knowledge base and that graph should be loaded with the content of the web 
resource "http://data.example.com/dataset1.ttl" that is protected with a basic http autentication

```
_:myCredentials a kees:BasicHttpAuthentication ;
	rdfs:label "reader credential"
	rdfs:comment "Please enter username and password for protected resources"
.
_:myAccrualPolicy a kees:UpdateGraphPolicy ;
	dct:accrualMethod [ a kees:BufferedUpload ;
		kees:requiresAuthentication _:myCredentials 
	]
.

[] a kees:ABoxGraph; dct:source <http://data.example.com/dataset1.ttl> ; dct:accrualPolicy _:myAccrualPolicy .
```

These triples MUST be equivalent to:

```
_:myCredentials a kees:BasicHttpAuthentication ;
	rdfs:label "reader credential"
	rdfs:comment "Please enter username and password for protected resources"
.
_:myAccrualPolicy a kees:UpdateGraphPolicy ;
	dct:accrualMethod [ a kees:BufferedUpload ;
		kees:requiresAuthentication _:myCredentials 
	] ;
	kees:onFetchingError [ a kees:RetentionPolicy ; kees:hasResilience 0 ] ;
.

[] a kees:ABoxGraph, sd:namedGraph;
	sd:name  <http://data.example.com/dataset1.ttl> ;
	dct:source <http://data.example.com/dataset1.ttl> ;
	dct:accrualPolicy _:myAccrualPolicy
.

[]	void:Dataset ;
	void:dataDump <http://data.example.com/dataset1.ttl>;
.

[] a qb:Observation ;
	daq:computedOn [ a sd:NamedGraph; sd:name <http://data.example.com/dataset1.ttl> ] ; 
    daq:metric kees:trustRank;
    daq:value "0.5"^^xsd:double;
	daq:isEstimated true .
.

```

A KEES compliant agent SHOULD get someway the username and password required by http basi authentication;
it can use rdfs:label and rdfs:comment in UI.
Than, using such credentials, check if <http://data.example.com/dataset1.ttl> resource is newer than the creationd date of the
<http://data.example.com/dataset1.ttl> named graph in the knowledge base.
If yes it SHOULD load the resouce in a temporary directory and load it in some way to the graph db 

### simple web resource incremental loading

This states that a n ABoxGraph SHOULD exists in the knowledge base and that graph should updated appending the content of the web 
resource "http://data.example.com/dataset1.ttl"

```
[] a kees:ABoxGraph; dct:source <http://data.example.com/dataset1.ttl> ; dct:accrualPolicy [ a kees:AppendDataPolicy ] .
```

These two RDF triples MUST be equivalent to:

```
[] a kees:ABoxGraph, sd:namedGraph;
	sd:name  <http://data.example.com/dataset1.ttl> ;
	dct:source <http://data.example.com/dataset1.ttl> ;
	dct:accrualPolicy [	
		a kees:AppendDataPolicy;
		dct:accrualMethod [ a kees:SparqlUpload ] 
	]
.

[]	void:Dataset ;
	void:dataDump <http://data.example.com/dataset1.ttl>;
.

[] a qb:Observation ;
	daq:computedOn [ a sd:NamedGraph; sd:name <http://data.example.com/dataset1.ttl> ] ; 
    daq:metric kees:trustRank;
    daq:value "0.5"^^xsd:double;
	daq:isEstimated true .
.

```

For example a KEES compliant agent SHOULD check if <http://data.example.com/dataset1.ttl> resource is newer than 
 <http://data.example.com/dataset1.ttl> named graph in the knowledge base; if yes it COULD execute the following sparql update statements:

```
LOAD <http://data.example.com/dataset1.ttl> INTO GRAPH <http://data.example.com/dataset1.ttl> ;
WITH <http://data.example.com/dataset1.ttl>
DELETE { ?g dct:modified ?anydate  }
INSERT  {
	?g  dct:modified "here current date^xsd:date ;
		prov:wasGeneratedBy [ 
			a prov:Activity , kees:SparqlUploadActivity ;
			prov:wasAssociatedWith [ a kees:Agent ] ;
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
WHERE {
	?g sd:name <http://data.example.com/dataset1.ttl> 
}
```


## Contributing to the site

A great way to contribute to the site is to create an [issue](https://github.com/linkeddatacenter/kees/issues) on GitHub when you encounter a problem or something. We always appreciate it. You can also edit the code by yourself and create a pull request.


All stuff here in the Creative Common (unless otherwise noted)
