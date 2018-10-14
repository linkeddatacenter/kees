KEES: Knowledge Exchange Engine Schema 
=======================================

**WARNING: WORKING IN PROGRESS**

In order to let computers to work for us, they must understand data. 
Not only grammar and syntax (like in EDI) but the real meaning of things. 
When this happens, machines can make decisions. Even when data are incomplete.
The same concerns apply to the knowledge itself: to share knowledge we need a language that is understandable both for humans and machines.

The KEES open specifications provide the tools to describe a knowledge base and to make it tradeable and shareable. KEES is a Semantic Web Application.

A.I. and humans can use these specifications to populate, merge and enrich a domain specific knowledge base. 

See [KEES project presentation](https://docs.google.com/presentation/d/1mv9XO0Q9QFxSphWzT_68Q4aXd9sgqWoY7njomH8eaPQ/pub?start=false&loop=false&delayms=5000)

## Core KEES concepts

Lot of concepts used by KEES refer to the well known [Semantic Web Standards](https://www.w3.org/standards/semanticweb/) published by the World Wide Web Consortium ([W3C](https://w3.org/)).

What is **data**? According with common sense, KEES defines data as words, numbers or in general any string of symbols.  This concept is equivalent to the definition of "literal" in the [RDF] (Resource Data Framework). Example of data is the string  `xyz`, the numbers `123`, `33.22` or the URI `http://LinkedData.Center`. Note that the data is usually associated with a  _data type_ it is just a name that states a set of restrictions on symbols string that build up the data;  _data type_ is not the data meaning.

What is **information**? KEES defines information as data with a meaning. The meaning can be 
learned from the context where a data is found or explicitly defined. From a practical point of view, because KEES adopts the [RDF] standards, an information is defined by three data that build up a _triple_ (also known as a RDF statement): a _subject_, a _predicate_ and an _object_. The data type for the first two triple elements (subject and predicate) must be an URIs, the last element (object) can be any data type.

KEES defines **knowledge** as a nework of linked information (i.e. linked data). This neworks is possible because, in RDF, any URI can be both the object of an information  and subject of another one or even apredicate for another.

KEES defines **knowledge base** as a container of linked information. From a more formal point of view, a _knowledge base_ is 
a semantic system where information is described as a set of statements according with the W3C standard 
Resource Description Framework (RDF). A  Knowledge Base is composed by two disjoint set of statements: *TBox* and *ABox*. 
*TBox statements* describe a system in terms of controlled vocabularies. TBox statements sometimes associate with object-oriented classes.
TBox statements tend to be more permanent within a knowledge base and are often grouped in "ontologies" that describe a specific 
knowledge domain (e.g. business entities, people, goods, friendship, offering, geocoding, etc, etc).
*ABox statements* associate with instances of classes defined by TBox statements. ABox statements are much more dynamic in nature and 
are populated from datasets available in the web or by reasonings. A knowledge base is often implemented with **Graph database**
and it is composed by the union of all REF triples contained in a set of disjoined Named Graphs.

The **KEES Language Profile** is  the vocabulary that describes the knowledge base content; more of less is the _TBox_ knowledge base partition. Any knowlege should contain a language profile.  _KEES Language Profile_ reuses some terms from existing
vocabularies, eventually adding some restrictions in term usage.

KEES Language Profile terms have been taken from the following vocabularies:

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


A **KEES Agent** is a semantic web agent that knows KEES language profile and that it is able to do actions on a 
knowledge base. It should be able to execute accrual policies and methods and to answer to a set of questions

A **Rule** describes how to generate/validate knowledge base statemensts using an algoritmic approach. 

An **Axiom** describes how to generate/validate knowledge base statemensts using a declarative approach. For instance an axiom is represented with
a sparql construct or with a SHACL restriction or with an entailment inferred by language profile semantic.

**Trust** is a key concept in KEES. The [RDF] model allow to mix any kind of information, even information that are incoerent with the language profile. For instace, suppose that your Language Profile contains a property "person:hasMom" that require a cardinality of 1 (i.e. every person has just one "mom"), your knowledge base could contains two different triples (:jack person:hasMom :Mary) and (:jack person:hasMom :Giulia), in order to take decision about who is jack's mom you need trust in your data. If you are sure about the veridicicy of all data in the knowledge base, you can deduct that :Mary and :Giulia are the same person. If you ar not so sure, you have two possibility: choose the most trusted statement with respect some criteria (eve casually if both statemenst have the same trust rank) or to change the language profile, allowing a person to have more than one mom. In any case you need to get an idea about _your_ trust on each statement (both ABox and Tbox) in the knowlege base.

Knowing the statement **provenance** is the most usefull way to get an idea about its trustability. For this reason, KEES requires that any statement must have a fourth element that links to metadata that describe any statement in the knowledge base. This means that, for pratical concerns, the KEES knowledge base is a collection of quads, i.e. a triple plus a link to a metadata

## The KEES vocabulary

The http://linkeddata.center/kees/v1#  namespace ( usual prefix *kees:*) contains a small set of concepts, mainly derived from existing ontologies. It provides terms to describe the knowledge base metadata that a KEES complain agent SHOULD/MUST know.

**kees:InferredDataGraph** states a named graph that contains only RDF statements derived from a reasoning.

**kees:LinkedDataGraph** states a named graph that contains only RDF statements derived from a source. 
The source MUST exists and MUST be referrenced as dcat:accessURL property in at least a dcat:Distribution of a dcat:Dastasets, part of a dset:Catalog. 
A KEES agent MUST recognize all mandatory properties defined in DCAT-AP for dcat:Catalog, dcat:Dataset and dcat:Distribution plus dct:modified 
property on dcat:Dataset.

a **kees:Report** states a parametric SPARQL query template that can be stored as  RDF data in a knowledge base. Eache report must provide an unique id (dct:identifier) in the knowledge base.

A **kees:Table** states a report that contains a SPARQL SELECT statement. 

A **kees:Document**  states a report that contains a SPARQL CONSTRUCT statement. 

A **kees:Question** states a report that contains a SPARQL ASK contained statement. 


Beside classes and properties, kees vocabulary defines a set of individuals:

- **kees:guard** a sparql service description feature that state the kees support (see below)

- **kees:trustMetric** defines a generic trust metric computed on an arbitrary requirements. I.e:

```
[] a qb:Observation ;
    daq:computedOn (:a_graph schema:LocalBusiness schema:legalName) ; 
    daq:metric kees:trustMetric ;
    daq:value 0.99 ;
    daq:isEstimated true .
```

- **kees:trustGraphMetric** defines a metric that evaluate a subjective trust value for a graph with a specific name. Can be used in graph quality observation. i.e.:

```
[] a qb:Observation ;
    daq:computedOn :a_graph ; 
    daq:metric kees:trustGraphMetric;
    daq:value 0.9 ;
    daq:isEstimated true .
```
The previous statements appy to all tripes contained in a named graph with `sd:name  :a_graph`

TODO: A formal definition of kees vocabulary is availabe as a [RDFS file](v1/kees.rdf).

TODO: KEES language profile restrictions is formally expressed in [SHACL constraints file](v1/kees-profile.rdf)

A set of other terms is currently on evaluation:

**kees:KBConfigGraph** states a linked data graph that contains statements that describes the knowledge base itself as a set of RDF triples.

**kees:Reasoning** is the process (i.e. an activity) of evaluating Rules and Axioms. The evaluation results can be stored results in the knowledge base (materializing) or just used by the query processor (e.g. SPARQL a endpoint)

**kees:TBoxGraph** states a linked data graph that contains TBOX statements.

A **kees:AccrualPolicy** states when a named graph SOULD be created or updated and what to do if there are errors in the acctual process.

A **kees:AccrualMethod** states what process to use to accrual data (e.g. an Exctract Trasform Load process) with all needed additional paramethers

## RDF Store requirement

Any RDF Store that provides with a SPARQL endpoint and QUAD support is compliant with KEES. Following requirement applies:

All KESS related information SHOULD be contained in a graph named <urn:graph:kees>

If a statement with subject <urn:kees:kb> and predicate dct:valid is present in the graph <urn:graph:kees>, this  means that the Knowledge base is in the *teaching window* windows (i.e. safe to be queried). Otherwhise the kees status of the knowledge base should be considered undefined.

For example: to declare that a RDF Store is ready to be safely queried execute following SPARQL UPDATE statement

```
WITH <urn:kees:config>
INSERT { <urn:kees:kb> <http://purl.org/dc/terms/valid> ?now }
WHERE { BIND( NOW() AS ?now) }
```

to check if a RDF Store is ready to be safely queried `ASK { <urn:kees:kb> <http://purl.org/dc/terms/valid> [] }`

## SPARQL endpoint requirements

A KEES compliant sparql endpoint SHOULD support following features:

- **kees:guard**: A KEES compliant sparql endpoint SHOULD return 503 Error of any SPARQL QUERY that happens on a RDF Store that ist not in the  *teaching window* state.  A KEES compliant sparql endpoint SHOULD should recognize the http header "X-KEES-guard: disable" in any SPARQL QUERY requests to disable *kees:guard* feature

A KEES compliant sparql endpoint SHOULD support http caching specs for all SPARQL queries that happened in the same teaching windows.


## KEES by examples

[** WARNING: THIS SECTION IS INFORMATIVE AND SUBJECTED TO CHANGE **]

### simple web resource (re)loading

This states that an graph named `:example`  SHOULD exist in the knowledge base and that graph should be loaded with the content of the web resource "http://data.example.com/dataset1.ttl"

```
[] a kees:LinkedDataGraph; sd:name :example; dct:source <http://data.example.com/dataset1.ttl> .
```
These two RDF triples are equivalent to:

```
[] a kees:LinkedDataGraph, kees:ABoxGraph, sd:NamedGraph;
	sd:name  :example ;
	dct:source <http://data.example.com/dataset1.ttl> ;
.

[]	dcat:Distribution, void:Dataset ;
	dcat:downloadURL <http://data.example.com/dataset1.ttl>;
	dct:format "text/turtle"
.
```

A KEES compliant agent implementation SHOULD check if <http://data.example.com/dataset1.ttl> resource is newer 
than  :example named graph in the knowledge base; 
if yes it COULD execute the following sparql update statements:

```
DROP SILENT GRAPH :example ;
LOAD <http://data.example.com/dataset1.ttl> INTO GRAPH :example ;
WITH <http://data.example.com/dataset1.ttl>
INSERT {
	?graph_description a kees:LinkedDataGraph; dct:modified ?now .
	# following properties COULD be inferred from http "Last-Modified:", "ETAG" and content-type headers
	<http://data.example.com/dataset1.ttl> 
		dct:modified  "2017-03-30"^^xsd:date ;
		dct:identifier "123456789" ;
		dct:format "text/turtle" .
}
WHERE {
	BIND( NOW() as ?now)
	?named_graph sd:name :example
}
```

**N.B.** the graph description URI MUST be different from sd:name .

### adding accrual info

It is possible to specify graph accrual method, but its semantic is left to agent implementation. e.g:

```
resource:graph_1 a kees:LinkedDataGraph;
	dct:source <http://data.example.com/dataset1.ttl> ;
	dct:accrualMethod ( ex:load_from_local_mirror "mirror/dataset1.ttl" "turtle" );
.
```

### adding accrual periodicity

A KEES compliant agent should be able to manage accrual periodicity. e.g:

```
resource:graph_1 a kees:LinkedDataGraph;
	dct:source <http://data.example.com/dataset1.ttl> ;
	dct:accrualPeriodicity <http://purl.org/linked-data/sdmx/2009/code#freq-W>
.
```

A KEES agent implementation SHOULD recognize at frequency instance from 
the [Content-Oriented Guidelines](http://www.w3.org/TR/vocab-data-cube/#dsd-cog) 
developed as part of the W3C Data Cube Vocabulary efforts. 


### adding trust info

Trust in dataset can be expessed with:

```
[] a qb:Observation ;
    daq:computedOn ?x ; 
    daq:metric kees:trustRank;
    daq:value 1.0 ;
    daq:isEstimated true 
.
```

If no explicit observation records are present in the knowledge base, this axiom SHOULD applies:

```
CONSTRUCT {
   [] a qb:Observation ;
      daq:computedOn ?x ; 
      daq:metric kees:trustRank;
      daq:value 0.5 ;
      daq:isEstimated true .
} WHERE {
      ?g sd:name ?name ;
      FILTER NOT EXISTS { ?observation daq:computedOn ?x  }
}
```


If more than an observation of a trust Rank is present, the average SHOULD be considered. As example see this axiom:

```
CONSTRUCT {
	?g kees:calculatedTrust ?trust 
} WHERE {
	?g sd:name ?name .
	{
		SELECT ?g (AVG(?value) AS ?trust)
		WHERE {
		  ?observation a qb:Observation ;
		  daq:metric kees:trustRank;
		  daq:computedOn ?g ;
		  daq:value ?value
		} GROUP BY ?g
	}
}
```

### simple protected dataset (re)loading
This states that an ABoxGraph named *http://data.example.com/dataset1* SHOULD exist in the knowledge base and that graph should be loaded with the content of the web 
resources *http://data.example.com/dataset1.ttl?page=1* and *http://data.example.com/dataset1.ttl?page=2* that are protected with a basic http autentication.

```
resource:graph_1 a kees:LinkedDataGraph; sd:name <http://data.example.com/graph/dataset> dct:source <http://data.example.com/dataset1.rdf> -
resource:distrib_1  dcat:accessURL <http://data.example.com/dataset1.rdf> ;
	void:dataDump 
	    <http://data.example.com/dataset1.ttl?page=1>,
        <http://data.example.com/dataset1.ttl?page=2>
.
resource:auth_1 a kees:BasicHttpAuthentication ;
	kees:realm "example.com private area";
	kees:uriRegexPattern "^http://data.example.com/dataset1.ttl"
.
```

A KEES compliant agent SHOULD get someway the username and password required by the http basic authentication; 
it could use rdfs:label and rdfs:comment to ask these data to user.
Than, using such credentials, check if <http://data.example.com/dataset1.rdf> resource is newer than the creation date of the
<http://data.example.com/graph/dataset> named graph in the knowledge base.
If yes it SHOULD load the two datadump resources in a cache and loads them in the same graph in kb.

### simple reasoning chain

This states that a some InferredKnowledgeGraph SHOULD created on a knowledge base when some facts changes 

```
resource:inference_1 a kees:InferredKnowledgeGraph ;
	sd:name graph:inferredAlternateNames ;
	dct:title "Inferred alternate names" ;
	dct:accrualMethod ( ex:eval_costructor <axioms/alternateNames.constructor> ). 

resource:inference_2
	a kees:InferredKnowledgeGraph ;
	sd:name graph:inferredLinksToCities ;
	dct:title "Inferred links to Cities"  ;
	dct:accrualMethod ( ex:sparql_update <axioms/linkCities.update> ).

resource:reasoning a kees:Reasoning ;
	kees:onEvent kees:facts_change ;
	kees:reasoningChain (
		resource:inference_1
		resource:inference_2
	) .

<axioms/alternateNames.constructor> dct:format "application/sparql-query".
<axioms/linkCities.update> dct:format "application/sparql-update".
```

Note that in previous example KEES agent implementation is supposed to understand  ex:eval_costructor and ex:sparql_update accrual methods.
The reasoningChain property allow to describe reasoning sequence.


### simple web resource incremental loading

This states that an ABoxGraph SHOULD exists in the knowledge base and that graph should updated weekly appending the content of the web 
resource "http://data.example.com/dataset1.ttl"

```
[
	a kees:LinkedDataGraph; 
	dct:source <http://data.example.com/dataset1.ttl> ;
	dct:accrualPeriodicity <http://purl.org/linked-data/sdmx/2009/code#freq-W>
	dct:accrualPolicy kees:append 
].
```

If some error occur during incremental append accrual policy, the graph is not updated else following changes to medatata SHOULD happen:

```
WITH <http://data.example.com/dataset1.ttl>  
DELETE { ?g dct:modified ?old }
INSERT { ?g dct:modified "here current date"^xsd:date}
WHERE { ?g sd:name <http://data.example.com/dataset.ttl> ; dct:modified ?old }
```

### simple web resource with custom accrual policy

This states that an ABoxGraph SHOULD exists in the knowledge base and that graph should created when a fact change. If errors occurs
during the accrual process, old data are retained but only till three consecutive errors, at forth failure of the accrual process
the graph is deleted.

```
[
	a kees:LinkedDataGraph; 
	dct:source <http://data.example.com/dataset.ttl> ;
	dct:accrualPolicy [ a  kees:AccrualPolicy;
		kees:onFetchingError [ a kees:RetentionPolicy ; kees:hasResilience 3 ] ;
	] 
]
```

On accrual error, the  following metadata SOULD be added to the named graph:

```
INSERT DATA {
	GRAPH <http://data.example.com/dataset.ttl> {
		[] a prov:Entity, sd:NamedGraph ; 
			sd:name <http://data.example.com/dataset1.ttl> ;
			prov:wasInvalidatedBy [ 
				a prov:Activity;
				prov:endedAtTime  "here the error date"^xsd:date ;
				prov:wasAssociatedWith kees:load_resource_processor ;
			] ;
	}
}
```

### simple web resource with custom accrual method

This states that an ABoxGraph SHOULD exists in the knowledge base and that graph should updated using a custom method that 
requires two additional parameters:

```
[
	a kees:LinkedDataGraph ;
	sd:name graph:dataset ;
	kees:accrualMethod ( sdaas:etl "csv1_gateway" "gateqay paramethers") ;
	dct:source <http://data.example.com/dataset1.csv>
] .
```

A KEES compliant agent SHOULD  know how to generate the graph named graph:dataset using <http://data.example.com/dataset1.csv> and the string "csv1_gateway".
The AGENT shoud generate at least the following metadata:

```
INSERT DATA {
	GRAPH graph:dataset {
		[] a prov:Entity, sd:NamedGraph ; 
			sd:name graph:dataset ;
			dct:modified "here current date"^xsd:date ;
			prov:wasGeneratedBy [ 
				a prov:Activity  ;
				prov:used [ a prov:Entity (sdaas:etl <http://data.example.com/dataset1.csv> "csv1_gateway") "gateqay paramethers"]
			] ;
		.
		# following properties COULD be inferred from http "Last-Modified:", "ETAG" and content type headers
		<http://data.example.com/dataset1.csv> a prov:Entity , foaf:Document ;
			dct:modified  "2017-03-30"^^xsd:date ;
			dct:identifier "123456721289" ;
		    dct:format "text/csv" ;
		.	
	}
}
```


## A complete example

See [examples directory](examples)



## Contributing to the site

A great way to contribute to the site is to create an [issue](https://github.com/linkeddatacenter/kees/issues) on GitHub when you encounter a problem or something. We always appreciate it. You can also edit the code by yourself and create a pull request.


All stuff here in the Creative Common (unless otherwise noted)


[RDF]: https://www.w3.org/TR/rdf11-primer/
