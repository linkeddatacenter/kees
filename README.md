KEES: Knowledge Exchange Engine Schema 
=======================================

**WARNING: WORKING IN PROGRESS**

In order to let computers to work for us, they must understand data: 
not just the grammar and the syntax but the real meaning of things. 

KEES proposes some specifications to add metadata to a *domain knowledge* in order to make it tradeable and shareable. 
Artificial Intelligences and humans can use KEES to populate, merge, exchange and enrich such knowledge. KEES is a Semantic Web Application.

See [KEES project presentation](https://docs.google.com/presentation/d/1mv9XO0Q9QFxSphWzT_68Q4aXd9sgqWoY7njomH8eaPQ/pub?start=false&loop=false&delayms=5000)

## Core KEES concepts

Lot of concepts used by KEES refer to the well known [Semantic Web Standards](https://www.w3.org/standards/semanticweb/) published by the World Wide Web Consortium ([W3C](https://w3.org/)).

What is **data**? According with common sense, KEES defines data as words, numbers or in general *any string of symbols*.  This concept is equivalent to the definition of "literal" in the [RDF] (Resource Data Framework). Example of data is the string  `xyz`, the numbers `123`, `33.22` or the URI `http://LinkedData.Center`. Note that the data is usually associated with a  _data type_ it is just a name that states a set of restrictions on symbols string that build up the data;  _data type_ is not the data meaning.

What is **information**? KEES defines information as *data with a meaning*. The meaning can be 
learned from the context where a data is found or explicitly defined. From a practical point of view, because KEES adopts the [RDF standards](https://www.w3.org/RDF/), an information is defined by three data that build up a _triple_ (also known as a RDF statement): a _subject_, a _predicate_ and an _object_. The data type for the first two triple elements (subject and predicate) must be an URIs subclass, the last element (object) can be any data type.

KEES defines **knowledge** as a nework of linked information (i.e. linked data). This nework is possible because, in RDF, any URI can be both the object of a triple  and the subject of another one or even a predicate for another.

KEES defines **knowledge base** as a container of related *linked data with a purpose*. 
From a more theoretical point of view, a _knowledge base_ is 
a semantic system where information is described as a set of statements according with the W3C standard 
Resource Description Framework (RDF). A  Knowledge Base is partitioned by two set of statements: *TBox* and *ABox*. *ABox statements* describe the facts,  *TBox statements* describe the terms used to qualify the facts. If you are familiar with object-oriented paradigm, TBox statements sometimes associate with classes, while ABox associate with individual class instances. 

*TBox statements* tend to be more permanent within a knowledge base and are often grouped in *ontologies* that describe a specific 
knowledge domain (e.g. business entities, people, goods, friendship, offering, geocoding, etc, etc).
*ABox statements* associate with instances of classes defined by TBox statements. ABox statements are much more dynamic in nature and 
are populated from datasets available in the web or by reasonings. 

For the KEES practical purposes, a knowledge base is a sublcass of a [service description dataset](https://www.w3.org/TR/sparql11-service-description/#sd-Dataset) 

A **knowledge graph** is an instance og a knowledge base implemented as [SPARQL service](https://www.w3.org/TR/sparql11-service-description/#sd-Service)
and it is composed by the union of all REF triples contained in a set of disjoined **Named Graphs**.

The **Language Profile** (or **Application profile**) is  the vocabulary that describes the knowledge that is recognized by a specific software application. The language profile is normally described in the Tbox partition of a knowledge base. Any application should refer to a language profile, that is an extension of the *data model* concept.  

A language profile is described by a set of formal vocabularies and some restrictions.

The **KEES Language Profile** is a language profile to describe a knowledge base itself. It reuses terms from existing vocabularies:

- dcat: http://www.w3.org/ns/dcat#
- dct: http://purl.org/dc/terms/
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

A **KEES application** is a Semantic Web Application that includes the KEES Language Profile, extending the vocabulary to describe a specific knowledge domain. 

KEES introduced some **new concepts**.

The most important concept introduced by KEES  is the **Key question**. 
Key questions represent the purpose for the the knowledge base existence. In other words, the knoledge base exists to answer to *key questions*. Key question are natural language expressions that can be expressed as parametric SPARQL queries on a populated knowledge graph. The answer to a key question can be a table of data, a structured document, a boolean or a translation of thes in a natural language sentence. [must add refences to http://inference-web.org works?]

A **KEES Agent** is a software process that understands a portion the language profile and that it is able to do actions on a 
knowledge base. For instance, it could be able to ingest data and to answer some questions.

A **Rule** describes how to generate/validate knowledge base statemensts using an algoritmic approach. 

An **Axiom** describes how to generate/validate knowledge base statemensts using a declarative approach. 
For instance an axiom is represented with a sparql construct or with a SHACL restriction or with an 
entailment inferred by language profile semantic.

**Trust** is another key concept in KEES. The [RDF] model allow to mix any kind of information, even information that are incoerent with the language profile. For instace, suppose that your Language Profile contains a property "person:hasMom" that require a cardinality of 1 (i.e. every person has just one "mom"), your knowledge base could contains two different triples (:jack person:hasMom :Mary) and (:jack person:hasMom :Giulia), in order to take decision about who is jack's mom you need trust in your data. If you are sure about the veridicicy of all data in the knowledge base, you can deduct that :Mary and :Giulia are the same person. If you are not so sure, you have two possibility: choose the most trusted statement with respect some criteria (even casually if both statemenst have the same trust rank) or to change the language profile, allowing a person to have more than one mom. In any case you need to get an idea about _your_ trust on each statement (both ABox and Tbox) in the knowlege base.

Knowing the statement **provenance** is the most usefull way to get an idea about its trustability. For this reason, KEES requires that any statement must have a fourth element that links to metadata that describe any statement in the knowledge base. This means that, for pratical concerns, the KEES knowledge base is a collection of quads, i.e. a triple plus a link to a metadata.

## The KEES vocabulary

The http://linkeddata.center/kees/v1#  namespace ( usual prefix *kees:*) contains a small set of concepts, mainly derived from existing ontologies. It provides terms to describe the knowledge base metadata that a KEES complain agent SHOULD/MUST know.

**kees:InferredDataGraph** states a named graph that contains only RDF statements derived from a reasoning.

[TO BE REVISED] **kees:LinkedDataGraph** states a named graph that contains only RDF statements derived from a data source. 
When possible, the source SHOULD be referrenced as dct:source property with a range dcat:DastaSet. The DataSet should expose following properties:

- *dct:modified*  with range a xsd:dateTime literal;
- *dct:accrualMethod* that points to the method to be used to ingest information in the knowwledge base; the method description is implementation dependent.
- *dct:accrualPeriodicity* that express the frequency with which items are added to the knowledge base;
- *dcat:distribution* that points to a dcat:Distribution object that represents web resource with a dcat:license property.


Beside classes and properties, kees vocabulary defines a set of individuals:

- **kees:guard** a [SPARQL service description](https://www.w3.org/TR/sparql11-service-description/#sd-Feature) feature that states that the RDF store supports KEES guard specifications (see below)

- **kees:trustMetric** defines a generic trust metric computed on an arbitrary requirements.

- **kees:trustGraphMetric** defines a metric that evaluate a subjective trust value for a graph with a specific name. Can be used in graph quality observations.

- **kees:sparqlQueryConstructOperation** states the datatype of a literal string containing a sparql query CONSTRUCT operation. 
- **kees:sparqlQuerySelectOperation** states the datatype of a literal string containing a sparql query SELECT operation.
- **kees:sparqlQueryDescribeOperation** states the datatype of a literal string containing a sparql query DESCRIBE operation.
- **kees:sparqlQueryAskOperation** states the datatype of a literal string containing a sparql query ASK operation.

TODO: A formal definition of kees vocabulary is availabe as a [RDFS file](v1/kees.rdf).

TODO: KEES language profile restrictions is formally expressed in [SHACL constraints file](v1/kees-profile.rdf)


**Here are a set of proposed terms to be discuss:**


**kees:KBConfigGraph** states a linked data graph that contains statements that describes the knowledge base itself as a set of RDF triples.


**kees:Reasoning** is the process (i.e. an activity) of evaluating Rules and Axioms. The evaluation results can be stored results in the knowledge base (materializing) or just used by the query processor (e.g. SPARQL a endpoint)

**kees:TBoxGraph** states a linked data graph that contains only TBOX statements.

**kees:AccrualPolicy** states when a named graph SOULD be created or updated and what to do if there are errors in the acctual process.

A **kees:AccrualMethod** states what process to use to accrual data (e.g. an Exctract Trasform Load process) with all needed additional paramethers


## The KEES workflow

A KEES workflow is based on a sequence of four temporal phases called "windows“:

- a startup  phase (**boot window**)  to initialize the knowledge base just with KEES description
- a time slot for the population of the Knowledge Base and to link data (**learning window**)
- a time slot for the data inference (**reasoning window**)
- a time slot to access the Knowledge Base and answering to questions  (**teaching window**)

KEES workflow is a continuous integration process. A guard SHOULD allow user to query the knowledge base only in the teaching windows.


## RDF Store requirement

Any RDF Store that provides with a SPARQL endpoint and QUAD support is compliant with KEES. Following requirement applies:

- All information retated too KEES booting window SHOULD be contained in a graph named <urn:graph:kees>
- If a statement with subject <urn:kees:kb> and predicate dct:valid is present in the graph <urn:graph:kees>, this  means that the Knowledge base is in the *teaching window* windows (i.e. safe to be queried). Otherwhise the kees status of the knowledge base should be considered undefined.

For example: to declare that a RDF Store is ready to be safely queried execute following SPARQL UPDATE statement

```
WITH <urn:graph:kees>
INSERT { <urn:kees:kb> <http://purl.org/dc/terms/valid> ?now }
WHERE { BIND( NOW() AS ?now) }
```

to check if a RDF Store is ready to be safely queried `ASK { <urn:kees:kb> <http://purl.org/dc/terms/valid> [] }`

## SPARQL service requirements

A KEES compliant sparql endpoint SHOULD support the  **kees:guard** feature: a KEES compliant sparql endpoint SHOULD return 503 Error of any SPARQL QUERY that happens on a RDF Store that is not in the  *teaching window* state.  A KEES compliant sparql endpoint SHOULD disable this feature if the http header "X-KEES-guard: disable" is present.

A KEES compliant sparql endpoint SHOULD support http caching specs [as described in Section 13 of RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html) for all SPARQL queries that happened in the same teaching windows.


## Examples

[** WARNING: THIS SECTION IS INFORMATIVE AND SUBJECTED TO MAYOR CHANGS **]



### TBD: Questions and Answers

```
[] a kees:Question
    dtc:identifier "question1" ;
    kees:hasParameter  "year" ;
    kees:answeredBySparqlQuery """
        PREFIX fr: <http://linkeddata.center/botk-fr/v1#>
        PREFIX qb:	<http://purl.org/linked-data/cube#>
        PREFIX interval: <http://reference.data.gov.uk/def/intervals/>
        PREFIX time: <http://www.w3.org/2006/time#>
        PREFIX ex: <http://example.org/app_data_model#>

        CONSTRUCT { 
            ?canonicalUri a ex:FinancialReport;
                ex:year ?reportYear ;
                ex:hasFact ?factUri.
            ?factUri ex:amount ?amount.
        }
        WHERE { 
            VALUES ?year { "2017" }
            
            ?financialReport a fr:FinancialReport; 
                fr:refPeriod/time:hasBeginning/interval:ordinalYear ?reportYear
            .
            ?financialFact a fr:Fact ;
                qb:dataSet ?finnacialReport ;
                fr:amount ?amount  ;
            .
            FILTER(STR(?reportYear)=?year)
            BIND( IRI(CONCAT("http://example.org/ldp/report/",?year)) AS ?canonicalUri)
            BIND( IRI(CONCAT("http://example.org/ldp/report/",?year,"/",STRUUID())) AS ?factUri)
        }
    """^^kees:sparqlQueryConstructOperation 
.
```

### TBD: simple web resource (re)loading

This states that an graph named `:example`  SHOULD exist in the knowledge base and that graph should be loaded with the content of the web resource "http://data.example.com/dataset1.ttl"

```
[] a kees:LinkedDataGraph; dct:source <http://data.example.com/dataset1> .
```
These two RDF triples are equivalent to:

```
[] a kees:LinkedDataGraph, kees:ABoxGraph, sd:NamedGraph;
	sd:name  <http://data.example.com/dataset1l> ;
	dct:source <http://data.example.com/dataset1> ;
.

<http://data.example.com/dataset1> a dcat:Dataset.
```

A KEES compliant agent implementation SHOULD check if there is a distribution resource that is newer 
than named graph in the knowledge base; 
if yes and the the distribution points to a download url http://data.example.com/dataset1.ttl>  it COULD execute the following sparql update statements:

```
DROP SILENT GRAPH :example ;
LOAD <http://data.example.com/dataset1.ttl> INTO GRAPH <http://data.example.com/dataset1.ttl> ;

INSERT {
	?named_graph a kees:LinkedDataGraph; dct:modified ?now .
	# following properties COULD be inferred from http "Last-Modified:" and "Content-type" resource header
	<http://data.example.com/dataset1.ttl> 
		dct:modified  "2017-03-30"^^xsd:date ;
		dct:format "text/turtle" .
}
WHERE {
	?named_graph sd:name  <http://data.example.com/dataset1l>.
	BIND( NOW() as ?now)
}
```

**N.B.** the graph description URI MUST be different from sd:name .

### TBD: adding accrual info

It is possible to specify graph accrual method, but its semantic is left to agent implementation. e.g:

```
resource:graph_1 a kees:LinkedDataGraph;
	dct:source <http://data.example.com/dataset1> ;
	dct:accrualMethod ( ex:load_from_local_mirror "mirror/dataset1.ttl" "turtle" );
.
```

### TBD: adding accrual periodicity

A KEES compliant agent should take into account accrual periodicity. e.g:

```
resource:graph_1 a kees:LinkedDataGraph;
	dct:source <http://data.example.com/dataset1> ;
	dct:accrualPeriodicity <http://purl.org/linked-data/sdmx/2009/code#freq-W>
.
```

A KEES agent implementation SHOULD recognize at frequency instance from 
the [Content-Oriented Guidelines](http://www.w3.org/TR/vocab-data-cube/#dsd-cog) 
developed as part of the W3C Data Cube Vocabulary efforts. 


### TBD: adding trust info

Trust in dataset can be expessed with:

```
[] a qb:Observation ;
    daq:computedOn (:a_graph schema:LocalBusiness schema:legalName) ; 
    daq:metric kees:trustMetric ;
    daq:value 0.99 ;
    daq:isEstimated true .
```

or

```
[] a qb:Observation ;
    daq:computedOn :a_graph ; 
    daq:metric kees:trustGraphMetric;
    daq:value 0.9 ;
    daq:isEstimated true .
```

If no explicit observation records are present in the knowledge base, this axiom SHOULD applies:

```
CONSTRUCT {
   [] a qb:Observation ;
      daq:computedOn ?g ; 
      daq:metric kees:trustGraphMetric;
      daq:value 0.5 ;
      daq:isEstimated true .
} WHERE {
      ?g sd:name ?name ;
      FILTER NOT EXISTS { ?observation daq:computedOn ?g  }
}
```


### TBD: simple reasoning chain

This states that a some InferredKnowledgeGraph SHOULD created on a knowledge base when some facts changes 

```
resource:inference_1 a kees:InferredKnowledgeGraph ;
	sd:name graph:inferredAlternateNames ;
	dct:title "Inferred alternate names" ;
	kees:runIf "ASK ..."^^kees:sparqlQueryAskOperation ; # inline condition
	dct:accrualMethod ( ex:eval_costructor <axioms/alternateNames.constructor> ). 

resource:inference_2
	a kees:InferredKnowledgeGraph ;
	sd:name graph:inferredLinksToCities ;
	dct:title "Inferred links to Cities"  ;
	kees:runIf <http://example.org/conditions/inference_condition.sparql_ask"> ; # application/sparql-query resource
	dct:accrualMethod ( ex:sparql_update <axioms/linkCities.update> ).

```

Note that in previous example KEES agent implementation is supposed to understand  ex:eval_costructor and ex:sparql_update accrual methods.
The reasoningChain property allow to describe reasoning sequence.


###  TBD: simple web resource incremental loading

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

###  TBD: simple web resource with custom accrual policy

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

###  TBD: simple web resource with custom accrual method

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



## Contributing to the site

A great way to contribute to the site is to create an [issue](https://github.com/linkeddatacenter/kees/issues) on GitHub when you encounter a problem or something. We always appreciate it. You can also edit the code by yourself and create a pull request.


All stuff here in the Creative Common (unless otherwise noted)


[RDF]: https://www.w3.org/TR/rdf11-primer/
