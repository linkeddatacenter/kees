Describing knowledge with KEES 
=======================================

**WARNING: WORKING IN PROGRESS**

In order to let computers to work for us, they must understand data: 
not just the grammar and the syntax but the real meaning of things. 

KEES (Knowledge Exchange Engine Service) proposes some specifications to add metadata to a *domain knowledge* in order to make it tradeable and shareable. 

KEES allows to formalize and license:

- how to collect the right data, 
- how much you can trust in your data, 
- what new information you can deduct from the collected data,
- how to answer to specific questions using data

Artificial intelligences and humans can use these *know hows* to reuse and enrich existing knowledge. KEES is a Semantic Web Application.

See [KEES presentation slides](https://docs.google.com/presentation/d/1mv9XO0Q9QFxSphWzT_68Q4aXd9sgqWoY7njomH8eaPQ/pub?start=false&loop=false&delayms=5000)

## Core KEES concepts

Lot of concepts used by KEES refer to the well known [Semantic Web Standards](https://www.w3.org/standards/semanticweb/) published by the World Wide Web Consortium ([W3C](https://w3.org/)).

What is **data**? According with common sense, KEES defines data as words, numbers or in general *any string of symbols*.  This concept is equivalent to the definition of "literal" in the [RDF] (Resource Data Framework). Example of data is the string  `xyz`, the numbers `123`, `33.22` or the URI `http://LinkedData.Center`. Note that the data is usually associated with a  _data type_ it is just a name that states a set of restrictions on symbols string that build up the data;  _data type_ is not the data meaning.

What is **information**? KEES defines information as *data with a meaning*. The meaning can be 
learned from the context where a data is found or explicitly defined. From a practical point of view, because KEES adopts the [RDF standards](https://www.w3.org/RDF/), an information is defined by three data that build up a _triple_ (also known as a RDF statement): a _subject_, a _predicate_ and an _object_. The data type for the first two triple elements (subject and predicate) must be an URIs, the last element of the triple (object) can be anything. A triple can be also rapresented as an [unidirected labeled graph]( :

KEES defines **knowledge** as a graph of linked information (i.e. linked data). This graph is possible because, in RDF, any URI can be both the object of a triple  and the subject of another one or even a predicate for another.

KEES defines **knowledge base** (or **knowledge graph** ) as a container of *linked data with a purpose*, that is a related set information that can be composed to provide answer to some questions. 

From a theoretical point of view, a KEES knowledge base is composed by information (i.e. fatcs), plus a formal system of logic used for knowledge representation, plus the [Open-world assumption](https://en.wikipedia.org/w/index.php?title=Open-world_assumption&oldid=871019791).

The information are partitioned in two set: *TBox* and *ABox*. *ABox statements* describe facts,  *TBox statements* describe the terms used to qualify the facts meaning. If you are familiar with object-oriented paradigm, TBox statements sometimes associate with classes, while ABox associate with individual class instances. 
*TBox statements* tend to be more permanent within a knowledge base and are often grouped in *ontologies* that describe a specific 
knowledge domain (e.g. business entities, people, goods, friendship, offering, geocoding, etc, etc).
*ABox statements* associate with instances of classes defined by TBox statements. ABox statements are much more dynamic in nature and 
are populated from datasets available in the web or by reasonings. 

For practical purposes, KEES assumes that the knowledge base can be defined in a [SPARQL service](https://www.w3.org/TR/sparql11-service-description).

The **Language Profile** (or **Application profile**) is the portion of the vocabularies (TBOX) that describe the knowledge that are recognized by a specific software application. The language profile contains *domain specific axioms* and is normally contained in the Tbox partition of a knowledge base. 

An **axiom** describes how to generate/validate knowledge base statemensts using entailment inferred by language profile semantic and known facts. For example an axiom can be described with OWL and evaluated by a OWL reasoner or described with SPARQL QUERY constructs or with SPARQL UPDATE scripts and evaluated in a SPARQL service.

**Trust** is another key concept in KEES. The [Open-world assumption] and RDF allow to mix any kind of information, even when information that are incoerent. For instance, suppose that your TBOX defines a property "person:hasMom" that require a cardinality of 1 (i.e. every person has just one "mom"), your knowledge base could contains two different fact (:jack person:hasMom :Mary) and (:jack person:hasMom :Giulia), in order to take decision about who is jack's mom you need trust in your data. If you are sure about the veridicicy of all data in the knowledge base, you can deduct that :Mary and :Giulia are two names for the same person. If you are not so sure, you have two possibility: choose the most trusted statement with respect some criteria (even casually if both statemenst have the same trust rank) or to change the TBOX , allowing a person to have more than one mom. In any case you need to get an idea about _your_ trust on each statement (both ABox and Tbox) in the knowlege base. At least you want to know the **provenance** and all metadata of all information in your knowledge base because the trust on a single data often derives from the trust of its source or in the creator of the data source.


## KEES Specification

## KEES Vocabulary

The **KEES vocabulary** defines few new terms the  http://linkeddata.center/kees/v1#  namespace ( usual prefix *kees:*). 
It consists of some OWL classes and properties, mainly derived from existing ontologies. 

All knowledge base building activities MUST be traced using PROV ontology. KEES requires that every build activity must be associated with a plan.

The main classes introduced by KEES vocabulary is the **kees:Plan**. A plan  describes how to create or update a named graph in the knowledge base extracting facts from a data source or from axioms.

A **kees:KnowledgeBase** is defined as a subclass of a sd:Dataset.

A **kees:KnowledgeBaseDescription** is a document to publishing and trasferring a KEES knowledge base bulding information.
Think it as a subclass of a foaf:Document. This allow to attach license and metadata to your knowledge base description,
thus making it sharable and tradeable.

The **kees:Question** represents the reason for the the knowledge base existence. In other words, the knoledge base exists to answer to *questions*. Question are natural language expressions that can be answered by a query on a populated knowledge graph. The answer to a question can result in tabular data, structured document, boolean sassertion or a translation of these in a natural language sentences.

A **kees:Agent** is a software process that understands a portion the KEES language profile and that it is able to do actions on a 
knowledge base. For instance, it could be able to ingest data and/or to answer some questions.

KEES vocabulary provide classes to describe reasoning that can be described as SPARQL scripts.

The KEES vocabulary is expressed with OWL RDF in [kees.rdf file](v1/kees.rdf). The file was edited with Protégé editor.

Besides few classes and properties, KEES vocabulary defines some individuals:

- **kees:guard** a [SPARQL service description](https://www.w3.org/TR/sparql11-service-description/#sd-Feature) feature that states that the RDF store supports KEES guard specifications (see below)
 **kees:trustGraphMetric** defines a metric that evaluate a trust value for a graph with a specific name. 
   All can be used in graph quality observations.
- **kees:append** and **kees:replace** state two possible graph accrual policies.
- If you want to share/trade your knowledge base, simply attach your KEES plan, questions, license and 
workflow to **kees:sharableKnowledge** object.

## The KEES Language profile

The **KEES Language Profile** reuses some terms from existing vocabularies adding mappings and restriction to the KEES vocabulary:

- dct: http://purl.org/dc/terms/
- sdmx-code: http://purl.org/linked-data/sdmx/2009/code#
- sd: http://www.w3.org/ns/sparql-service-description#
- foaf: <http://xmlns.com/foaf/0.1/>
- prov: http://www.w3.org/ns/prov#
- vann:<http://purl.org/vocab/vann/>
- voaf:<http://purl.org/vocommons/voaf#>
- spin: <http://spinrdf.org/spin#>

The following picture sumarizes the KEES language profile.

![uml](architecture/uml.png)


Trustability is enabled by attaching quality observation to ingested graph or to specific subset of statements. 
For this feature KEES suggest to reuse the [Dataset Quality Vocabulary (daQ)](http://purl.org/eis/vocab/daq) that requires
additional vocabularies:

- qb: http://purl.org/linked-data/cube#
- daq: http://purl.org/eis/vocab/daq# 

TODO: KEES language profile restrictions is formally expressed in [SHACL constraints file](v1/kees-profile.rdf)

A **KEES compliant application** is a Semantic Web Application that is not in conflict with the KEES Language Profile. 



## RDF Store requirement

Knowing the statement **provenance** is of paramount imortance to get an idea about its trustability. For this reason, KEES requires that any statement must have a fourth element that links to metadata that describe any statement in the knowledge base. This means that, for pratical concerns, the KEES knowledge base is a collection of quads, i.e. a triple plus a link to a metadata.

Any RDF Store that provides with a SPARQL endpoint and QUAD support is compliant with KEES. 

During knowledge base building and update the knowledge base could be in an inconsistent state.
To be sure that the knowledge base is in a consistent state, following convention applies:
if a statement with subject <urn:kees:kb> and predicate dct:valid is present in the default graph, this  means that the Knowledge base is *safe* to be queried. Otherwhise the knowledge base should be considered *not safe*.

For example: 

to declare that a RDF Store is ready to be safely queried execute following SPARQL UPDATE statement

```
INSERT { <urn:kees:kb> <http://purl.org/dc/terms/valid> ?now }
WHERE { BIND( NOW() AS ?now) }
```
To declare that a RDF Store is *not safe*

```
Delete { <urn:kees:kb> <http://purl.org/dc/terms/valid> ?x  } WHERE { <urn:kees:kb> <http://purl.org/dc/terms/valid> ?x }
```

To check if a RDF Store is ready to be safely queried `ASK { <urn:kees:kb> <http://purl.org/dc/terms/valid> [] }`


## SPARQL service requirements

A KEES compliant sparql endpoint SHOULD expose the  **kees:guard** feature. If a kees:guard feature is present
the endpoint MUST return 503 Error of any SPARQL QUERY that happens on a RDF Store that is not in the  *safe* state.  A KEES compliant sparql endpoint SHOULD disable this feature if the http header "X-KEES-guard: disable" is present in the request.

A KEES compliant SPARQL service SHOULD alwais provide proper http caching information [as described in Section 13 of RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html).


## KEES agent requirements

### Workflow

A KEES Agent SHOULD perform actions on a knowledge base on a sequence of four temporal phases called *windows*:

1. a startup  phase (**boot window**)  to initialize the knowledge base strating from one or more knowledge base descriptions
2. a time slot for the population of the Knowledge Base and to link data (**learning window**)
3. a time slot for the data inference (**reasoning window**)
4. a time slot to access the Knowledge Base and answering to questions  (**teaching window**)

Step 2 and 3 can be iterated.

This sequence is called **KEES workflow** and it is a continuous integration process that happens on scheduled time or 
after triggering an event (e.g. a dataset change).

One or more **kees:workflow** property SHOULD be attached to the default dataset of the service. A workflow points to a rdf
list of kees:Plan objects.  The plan execution order is strictly determined by the plan position in the list.

If more than a workflow is present, the execution order of workflow is implementation dependent. 
A KEES agen can decide to execute different workflow in parallel.


### Vocabulary knowledge

A KEES agent MUST recognize void:vocabulary property in a knowledge base. The object referred by the void:vocabulary property MUST point to a vocabulary defined as a voaf:Vocabulary that, in turn, exposes at least a vann:preferredNamespacePrefix propery.

A KEES agent MUST known at least following vocabularies:

- dct: http://purl.org/dc/terms/
- sdmx-code: http://purl.org/linked-data/sdmx/2009/code
- sd: http://www.w3.org/ns/sparql-service-description
- foaf: http://xmlns.com/foaf/0.1/
- prov: http://www.w3.org/ns/prov
- void: http://rdfs.org/ns/void
- vann: http://purl.org/vocab/vann/
- voaf: http://purl.org/vocommons/voaf
- spin:  http://spinrdf.org/spin
- rdf:  http://www.w3.org/1999/02/22-rdf-syntax-ns
- rdfs:	http://www.w3.org/2000/01/rdf-schema
- owl: http://www.w3.org/2002/07/owl
- kees: http://linkeddata.center/kees/v1
- dct: http://purl.org/dc/terms/
- dcterms: http://purl.org/dc/terms/

A KEES agent MUST recognize void:uriSpace property in a knowledge base. The  string referred by the void:uriSpace property MUST
be used to generate the default namespace.


### Graph building rules rules 

KEES does not impose any reasoner but a KEES agent MUST support at least some feature defined in the
[SPIN W3C Member Submission 22 February 2011, updated 07 November, 2014](http://spinrdf.org/spin.html) and in particular:

- the range the *kees:from* property SHOULD accept a type sp:Construct or sp:Update that exposes a non empty sp:text property.
- the range the *kees:assert* property SHOULD accept a type sp:Construct or sp:Ask that exposes a non empty sp:text property.
- the range the *kees:answerMethod* property SHOULD accept a type sp:Select or sp:Ask or sp:Construct that exposes a non empty sp:text property.

In sp:text is possible to use all prefixes defined in the knowledge base without need to declare a prefix. The same for the
default prefic.
If you declare a prefix in the SPARQL rule text, it will have the prioriry on vocabulary definition.


If the *kees:from* is not recognized,  at least, the agent is expected to execute perform like the SPARQL UPDATE statement "LOAD <here
the URI of kees:from property> INTO <here the URI of kees:builds property>" construct.


### Error conditions and error management

A KEES agent MUST update the RDF store safe statement when it enters or exits the teaching window. 

If a KEES agent was unable to complete succesfully a plan, it MUST abort if the target named graph was partially builded, otherwhise
it MUST annotate the named graph with the prov:InvalidatedAtTime property and continue.

If the KEES agent abort its execution, the knowledge base MUST resulting in a "not safe" state.

In normal operations, the existence of prov:InvalidatedAtTime in a named graph MUST prevent the KEES agent to enter the teaching window.
A KEES agent COULD provide a way to force a safe state when some named graph contains a prov:InvalidatedAtTime property.

The existence of prov:InvalidatedAtTime in SHOULD be signaled by KEES agent. How to signal is implementation dependent

The existence of activities without a plan SHOULD be signaled by KEES agent. This condition does not prevent the 
KEES agent to enter the teaching window.

A KEES agent MUST abort in case of semantic inconsistences in *KEES knowledge base definition*. KEES does not impose
restriction on knowledge base semanti consistence apart from its definition.

### Axioms

A KEES agent MUST ensure some  axioms before any plan execution.

If exists a plan  without kees:from fields, it MUST be generated from the kees:build property; e.g.:

```
CONSTRUCT { ?plan kees:from ?graphUri } 
WHERE {
   ?plan kees:build ?graphUri .
   FILTER NOT EXISTS { ?plan  kees:from [] }
}
```

Every plan expose exactly one kees:build property; e.g. this query MUST return true:

```
ASK {
   ?plan kees:build ?graphUri1 , ?graphUri2 .
   FILTER NOT EXISTS { ?graphUri1 != ?graphUri2 }
}
```

*kees:bulds* and *kees:required* referenced object must be interpreted as the sd:name of a named graph (existing or to be created)

*kees:builds* must be recognized as functional inverse property.

Only one instance of *kees:accrualPeriodicity* MUST exists.


If  kees:dataSource attribute is not present a KEES agent MUST reuse  kees:builds value, e.g.:

```
CONSTRUCT { ?plan kees:dataSource ?g } 
WHERE {
   ?plan kees:builds ?g
   FILTER NOT EXISTS { ?plan kees:dataSource ?g }
}
```

If no *kees:accrualPolicy* exists, kees:replace is used:


```
CONSTRUCT { ?plan kees:accrualPolicy kees:replace} 
WHERE {
   FILTER NOT EXISTS { ?plan kees:accrualPolicy [] }
}
```

Exactly one *kees:accrualPolicy* must be present.


## Plans execution process

A KEES agent MUSt implement a this process schema in plan executing:

1. execute default axioms
2. ensure the integrity of the knowledge base descriptions. Abort if errors.
3. if the target graph existsm use kees:accrualPeriodicity to define if the plan can be skipped.  How to evaluate this condition is implementation dependent.
4. test that at least one kees:required named graph is newer tha the building graph. If no kees:required propery exists eval to true
   if all named graph referenced by the *kees:required* properties are older than the target graph, the plan SHOULD be skipped.
5. Decide and take the appropriate PLAN action looking
6. if exist, evaluate all assert conditions, aborting if one return false

The KEES agent SHOULD ignore a plan if the named graph exists and points to da dataset that is older than all kees:from properties
How to evaluate this condition is implementation dependent but a data source without a clear modification date MUST 
be considered always newer than any existing named graph.

The KEES agent SHOULD recognize all concepts in sdmx-code:freq scheme for dct:accrualPeriodicity and use these information to
decide if executing a plan or not. How to manage dct:accrualPeriodicity depends from KEES Agent implementation.


A possible inmplementation on an algoritm that decide if a kees:requierement is satisfied :

```
ASK { 
   VALUES ?objectUri { <here the object URI of the  conditional statement> }
   VALUES ?subjectUri { <here the subject URI of the  conditional statement > }
   
   ?objectUri dct:modified ?objectModified.
   ?subjectUri dct:created ?subjectCreated.
   
   FILTER( ?objectModified > ?subjectCreated )
}
```


### KEES agent protocol

A KEES Agent MUST be able to accept in input one or more URL dereferencing to kees:KnowledgeBaseDescription resources.
The input method is implentation dependent. A KEES agent can be implemented as a web service or as a command or as a job.

There is no direct way to detect if an agent is running or is aborted. But you can always check if the knowledge base is in a
safe state. 

KEES agent implementation SHOULD add some log and monitor features.

KEES agent should be able to infer types from functional properties.

### Accrual policies

A KEES agent MUST recognize *kees:append* and *kees:replace* individuals as valid object for kees:accrualPolicy property in a kees:Plan. In case of inconsistencies or if no dct:accrualPolicy property is specified, the agent MUST choose kees:create.

If the accrual policy is kees:create, the named graph and all related metadata MUST be deleted and recreated BEFORE
to execute accrual policies.

### Accrual methods

A KEES agent should try to load all required resources specified by dct:requires property, in the named graph specified in
kees:builds property taking into consideration the resource types and accual methods.
	
If the dct:accrualMethod is a string with datatype *kees:sparqlQueryConstructOperation* then the agent MUST evaluate it
and add the constructed triples added to the named graph pointed by the plan

If the dct:accrualMethod is a string with datatype *kees:sparqlUpdate* then then the agent MUST evaluate it.


## Example

[** WARNING: THIS SECTION IS INFORMATIVE AND SUBJECTED TO MAYOR CHANGS **]


## Publishing knowledge base description as linked data resource

Create a file kees.ttl fit following content.

```
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix dct: <http://purl.org/dc/terms/> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .      
@prefix kees: <http://linkeddata.center/kees/v1#> .

<> a kees:KnowledgeBaseDescription; 
	dct:title "This dummy knowledge base is about teenagers in Europe"@en;
	dct:creator <https://example.org/profile/card#organization>;
	dct:license <http://www.opendatacommons.org/odc-public-domain-dedication-and-licence/> .
	foaf:primaryTopic kees:sharableKnowledge
.

```



## Defining TBOX vocabularies

```
kees:sharableKnowledge
	void:vocabilary 
		<http://schema.org/> , 
		<http://dbpedia.org/ontology/> ;
	void:uriSpace <http://example.org/resource/> ;
.

<http://schema.org/> a voaf:Vocabulary;
	vann:preferredNamespacePrefix "schema"
.
<http://dbpedia.org/ontology/> a voaf:Vocabulary;
	vann:preferredNamespacePrefix "dbo"
.

```


## Defining a workflow

Add to kees.ttl file:

```
kees:sharableKnowledge kees:workflow (
	[ kees:builds <http://data.example.com/peoples_from_europe.rdf> ]		
	[ 	a kees:ReasoningPlan;
		dct:title "Compute if a people shoud to be considered a teenager"@en;
		kees:builds :inference1 ;
		kees:requires <http://data.example.com/peoples_from_europe.rdf>; 	
		kees:from [ a sp:Construct; sp:text """
			CONSTRUCT { ?person a :class_teenager>  }
			WHERE {
				?person dbo:birthDate ?birth .
				BIND( NOW() - ?birth AS ?age)
				FILTER (?age > 11 && ?age < 20 )
			}
		"""]	
	]
) .
	
```

### Attaching questions


Add to kees.ttl file:

```
kees:sharableKnowledge kees:answers [  a kees:Question
	dct:title "Teenagers born in Berlin"@en;
	kees:answerMethod [ a sp:Ask; sp:text "SELECT DISTINCT ?person WHERE {?person a :class_teenager>}"]
 ].
```

### adding trust info


Trust in dataset can be expessed with:

```
[] a qb:Observation ;
    daq:computedOn <http://data.example.com/peoples_from_europe.rdf> ; 
    daq:metric kees:trustGraphMetric;
    daq:value 0.98 ;
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

## Contributing to the site

A great way to contribute to the site is to create an [issue](https://github.com/linkeddatacenter/kees/issues) on GitHub when you encounter a problem or something. We always appreciate it. You can also edit the code by yourself and create a pull request.


[Open-world assumption]: https://en.wikipedia.org/w/index.php?title=Open-world_assumption&oldid=871019791

All stuff here in the Creative Common (unless otherwise noted)


[RDF]: https://www.w3.org/TR/rdf11-primer/
