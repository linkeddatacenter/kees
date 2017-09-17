KEES: Knowledge Exchange Engine Schema 
=======================================

**WARNING: WORKING IN PROGRESS**

In order to let computers to work for us, they must understand data. 
Not only grammar and syntax (like in EDI) but the real meaning of things. 
When this is clear, machines can recognize, manage data and make decisions even when their provenance and 
quality is not totally known and data are incomplete.
The same concerns apply to the knowledge itself: to share knowledge we need a language that is understandable both for humans and machines.

This is the source code repository for the KEES language profile specifications that is used to describe a knowledge configurations.

Semantic web agents and humans can use these specificationa to populate, merge and share a domain specific knowledge base. 

See [KEES project presentation](https://docs.google.com/presentation/d/1mv9XO0Q9QFxSphWzT_68Q4aXd9sgqWoY7njomH8eaPQ/pub?start=false&loop=false&delayms=5000)


## Terminology used in the KEES language profile

The **KEES Language Profile** is a specification that re-uses terms from one or more base
standards, adding more specificity by identifying mandatory, recommended and
optional elements to be used for describing a knowledge base, as well as recommendations
for controlled vocabularies to be used.

A **KEES Agent** is a semantic web agent that knows KEES language profile and that it is able to do actions on a 
knowledge base taking into account the statements contained in all configuration graphs. It  SHOULD be able to execute accrual policies and methods and to 
answer to a set of questions

A **Rule** describes how to generate/validate knowledge base statemensts using an algoritmic approach. For instance  a rule could be implemented with some
sparql update insert statements or with a program.

An **Axiom** describes how to generate/validate knowledge base statemensts using a declarative approach. For instance an axiom is represented with
a sparql construct or with a SHACL restriction or with an entailment inferred by language profile semantic.

The KEES Language Profile reuses terms from various existing specifications. Classes and
properties have been taken from the following namespaces:

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

The **kees:** namespace define a small set of concepts, mainly derived from existing ontologies:

A **kees:KnowledgeBase** is a semantic system where information is described as a set of statements according with the W3C standard 
Resource Description Framework (RDF). A  Knowledge Base is composed by two disjoint set of statements: *TBox* and *ABox*. 
*TBox statements* describe a system in terms of controlled vocabularies. TBox statements sometimes associate with object-oriented classes.
TBox statements tend to be more permanent within a knowledge base and are often grouped in "ontologies" that describe a specific 
knowledge domain (e.g. business entities, people, goods, friendship, offering, geocoding, etc, etc).
*ABox statements* associate with instances of classes defined by TBox statements. ABox statements are much more dynamic in nature and 
are populated from datasets available in the web or by reasonings. A knowledge base is implemented as a **Graph database**
and it is composed by the union of all REDF triples contained in a  set of disjoined Named Graphs.



**kees:Reasoning** is the process (i.e. an activity) of evaluating Rules and Axioms. The evaluation results can be stored results in the knowledge base (matherializing)
or just used by the query processor (e.g. SPARQL a endpoint)

**kees:InferredDataGraph** states a named graph  that contains only RDF statements derived from a reasoning.

**kees:LinkedDataGraph** states a named graph that contains only RDF statements derived from a source. The source MUST exists and MUST be 
referrenced as  dcat:accessURL property in at least a dcat:Distribution of a dcat:Dastasets, part of a dset:Catalog. 
A KEES agent MUST recognize all mandatory properties defined in DCAT-AP for dcat:Catalog, dcat:Dataset and dcat:Distribution plus dct:modified 
property on dcat:Dataset.

**kees:KBConfigGraph** states a linked data graph that contains statements that describes the knowledge base itself as a set of RDF triples.

**kees:TBoxGraph** states a linked data graph that contains TBOX statements.

A **kees:AccrualPolicy** states when a named graph SOULD be created or updated and what to do if there are errors in the acctual process.

A **kees:AccrualMethod** states what process to use to accrual data (e.g. an Exctract Trasform Load process) with all needed additional paramethers

A **kees:Table** states a parametric sparql select contained in the knowlwdge base. A table often used to provide an answer to a question.


Beside classes and properties, kees vocabulary defines a set of individuals:

- **kees:facts_change** defines an event that occurs when at least a graph changed its content after a learning windows 
- **kees:trust_rank** defines a metric that evaluate a subjective trust value for a graph. Can be used in grapph quality observation. ie:

```
[] a qb:Observation ;
	daq:computedOn resource:a_graph ; 
    daq:metric kees:trust_rank;
    daq:value 0.9 ;
	daq:isEstimated true .
```

TODO: A formal definition of kees vocabulary is availabe as a [RDFS file](v1/kees.rdf).

TODO: KEES language profile restrictions is formally expressed in [SHACL constraints file](v1/kees-profile.rdf)


## KEES by examples

### simple web resource (re)loading

This states that an ABoxGraph SHOULD exist in the knowledge base and that graph should be loaded with the content of the web 
resource "http://data.example.com/dataset1.ttl"

```
resource:graph_1 a kees:LinkedDataGraph; dct:source <http://data.example.com/dataset1.ttl> .
```
These two RDF triples are equivalent to:

```
resource:graph_1 a kees:LinkedDataGraph,kees:ABoxGraph, sd:NamedGraph;
	sd:name  <http://data.example.com/dataset1.ttl> ;
	dct:source <http://data.example.com/dataset1.ttl> ;
	dct:accrualPolicy kees:reload_on_source_change ;
.

[]	dcat:Distribution, void:Dataset ;
	dcat:accessURL <http://data.example.com/dataset1.ttl>
	void:dataDump <http://data.example.com/dataset1.ttl>
.
```

A KEES compliant agent implementation SHOULD check if <http://data.example.com/dataset1.ttl> resource is newer 
than  resource:graph_1 named graph in the knowledge base; 
if yes it COULD execute the following sparql update statements:

```
DROP SILENT GRAPH <http://data.example.com/dataset1.ttl> ;
LOAD <http://data.example.com/dataset1.ttl> INTO GRAPH <http://data.example.com/dataset1.ttl> ;
INSERT_DATA  {
	GRAPH <http://data.example.com/dataset1.ttl> {
		resource:graph_1 dct:modified "here current date"^xsd:date .
		# following properties COULD be inferred from http "Last-Modified:", "ETAG" and content-type headers
		<http://data.example.com/dataset1.ttl> 
			dct:modified  "2017-03-30"^^xsd:date ;
			dct:identifier "123456789" ;
		    dct:format "text/turtle" 
		.	
	}
}
```

**N.B.** the graph description uri MUST be different from sd:name and MUST NOT be a blank node.

### adding accrual info

It is possible to specify graph accrual method, but its semantic is  left to agent implementation. e.g:

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

A KEES agent implementation SHOULD recognize at frequency instance from the [Content-Oriented Guidelines](http://www.w3.org/TR/vocab-data-cube/#dsd-cog) 
developed as part of the W3C Data Cube Vocabulary efforts. 


### adding trust info

Trust in dataset can be expessed with:

```
[] a qb:Observation ;
	daq:computedOn [ a sd:NamedGraph; sd:name <http://data.example.com/dataset1.ttl> ] ; 
    daq:metric kees:trustRank;
    daq:value 1.0 ;
	daq:isEstimated true 
.
```

If no explicit observation records are present in the knowledge base, this axiom SHOULD applies:

```
CONSTRUCT {
	[] a qb:Observation ;
		daq:computedOn ?g ; 
	    daq:metric kees:trustRank;
	    daq:value 0.5 ;
		daq:isEstimated true .
} WHERE {
	?g sd:name ?name ;
	OPTIONAL { ?observation daq:computedOn ?g  }
	FILTER( !BOUND(?observation))
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

A KEES compliant agent SHOULD get someway the username and password required by the http basic authentication; it could use rdfs:label and rdfs:comment to ask these data to user.
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

resoure:reasoning_1 a kees:ReasoningActivity ;
	keees:onEvent kees:facts_change ;
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
