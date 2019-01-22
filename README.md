Describing knowledge with KEES 
=======================================

**WARNING: WORKING IN PROGRESS**

In order to let computers to work for us, they must understand data: 
not just the grammar and the syntax, but the real meaning of things. 

KEES (Knowledge Exchange Engine Service) proposes some specifications to describe a *domain knowledge* in order to make it tradeable and shareable. 

KEES allows to formalize and license:

- how to collect the right data, 
- how much you can trust in your data, 
- what new information you can deduct from the collected data,
- how to answer to specific questions using data

Artificial intelligences and humans can use these *know hows* to reuse and enrich existing knowledge. KEES is a Semantic Web Application.

See [KEES presentation slides](https://docs.google.com/presentation/d/1mv9XO0Q9QFxSphWzT_68Q4aXd9sgqWoY7njomH8eaPQ/pub?start=false&loop=false&delayms=5000)

## Definitions

Lot of concepts used by KEES refer to the well known [Semantic Web Standards](https://www.w3.org/standards/semanticweb/) published by the World Wide Web Consortium ([W3C](https://w3.org/)).

What is **data**? According with common sense, KEES defines data as words, numbers or in general *any string of symbols*.   Example of data is the strins  `xyz`, `123`, `33.22` or  `http://LinkedData.Center`. Data is usually associated with a  _data type_ that states a set of restrictions on symbols dequence that build up the data For example:  the number `123`, the float `33.22` or  the URI `http://LinkedData.Center`. The _data type_ is not the data meaning.

What is **information**? KEES defines information as *data with a meaning*. The meaning can be 
learned from the context where a data is found or explicitly defined. KEES adopts the [RDF standards](https://www.w3.org/RDF/) to
describe information by tuple of three data, i.e. a _triple_ (also known as a RDF statement): a _subject_, a _predicate_ and an _object_. The data type for the first two elements of a triple (i.e. the subject and the predicate) must be an URIs, the last element of the triple (i.e. the object) can be anything. A triple can be also represented as an [unidirected labeled graph](https://mathinsight.org/definition/undirected_graph)

![a triple](architecture/triple.jpg)

KEES defines **knowledge** as a graph of linked information (i.e. linked data). This graph is possible because, in RDF, any URI can be both the object of a triple  and the subject of another one or even the predicate for another.

![triples](architecture/triples.png)

KEES defines **knowledge base** (or **knowledge graph** ) as a container of *linked data with a purpose*, that is a related set information that can be queried to provide answers to some questions. 

From a theoretical point of view, a knowledge base is composed by information (i.e. fatcs), plus a formal system of logic used for knowledge representation, plus the [Open-world assumption](https://en.wikipedia.org/w/index.php?title=Open-world_assumption&oldid=871019791), plus an inference engine that demonstrates theorems. 

The information are partitioned in two set: *TBox* and *ABox*. *ABox statements* describe facts,  *TBox statements* describe the terms used to qualify the facts meaning. If you are familiar with object-oriented paradigm, TBox statements sometimes associate with classes, while ABox associate with individual class instances. 
*TBox statements* tend to be more permanent within a knowledge base and are often grouped in *ontologies* that describe a specific 
knowledge domain (e.g. business entities, people, goods, friendship, offering, geocoding, etc, etc).
*ABox statements* associate with instances of classes defined by TBox statements. ABox statements are much more dynamic in nature and 
are populated from datasets available in the web or by reasonings. 

KEES assumes that RDF is used for knowledge representation  and that the knowledge base is implemented 
as a dataset in a [SPARQL service](https://www.w3.org/TR/sparql11-service-description).

The **Language Profile** (or **Application profile**) is the portion of the TBOX that is recognized by a specific software application.

The language profile can contain *axioms* . An **axiom** describes how to generate/validate knowledge base statemensts using entailment inferred by language profile semantic and known facts. For example an axiom can be described with OWL and evaluated by a OWL reasoner or described with SPARQL QUERY constructs or with SPARQL UPDATE scripts and evaluated in a SPARQL service.

The **KEES Language Profile** is the set of all terms, rules and axioms that a software application that want to use a knowledge base should to understand.

A **KEES Agent** describes a processor that understands the *KEES language profile*  and that it is able to do 
actions on a knowledge base  according with KEES specifications.
It should be able to learn data, reasoning about data and to answer some questions starting from learned fact.

**Trust** is another key concept in KEES. The [Open-world assumption] and RDF allow to mix any kind of information, even when information that are incoerent. For instance, suppose that an axiom in your knowledge base TBOX states that a property "person:hasMom" has a cardinality of 1 (i.e. every person has just one "mom"), your knowledge base could also contains two different facts (:jack person:hasMom :Mary) and (:jack person:hasMom :Giulia), peraphs extracted from different datasources. In order to take decision about who is jack's mom you need trust in your data. If you are sure about the veridicicy of all data in the knowledge base, you can deduct that :Mary and :Giulia are two names for the same person. If you are not so sure, you have two possibility: deduct that the data source is wrong, so you have to choose the most trusted statement with respect some criteria (even casually if both statemenst have the same trust rank) or to change the axiom in TBOX , allowing a person to have more than one mom. In any case you need to get an idea about _your_ trust on each statement, both in ABox and in Tbox,  in the knowlege base. At least you want to know the **provenance** and all metadata of all information in your knowledge base because the trust on a single data often derives from the trust of its source or in the creator of the data source.

## KEES Vocabulary

The **KEES vocabulary** defines few new terms in the  http://linkeddata.center/kees/v1#  namespace ( usual prefix *kees:*). 
It consists of some OWL classes and properties, mainly derived from existing ontologies. 

![uml](v1/images/uml.png)

A **kees:KnowledgeBase** is defined as a subclass of a [sd:Dataset](https://www.w3.org/TR/sparql11-service-description/#sd-Dataset) that, in turn, is a specialization of  a dataset as described in the [VoID](https://www.w3.org/TR/void/). A kees:KnowledgeBase is
a collection of [sd:namedGraph](https://www.w3.org/TR/sparql11-service-description/#sd-NamedGraph) that contain the linked data.

The main class introduced by KEES vocabulary is the **kees:Plan** that describes how to create and update a named graph in the knowledge base, extracting facts extracted from a data source or derived from axioms. 

A **kees:KnowledgeBaseDescription** is a document that contains the description of the knowledge base with the purpose of
publishing and trasferring  knowledge base bulding information.
It is a a subclass of a [foaf:Document](http://xmlns.com/foaf/spec/#term_Document) that allows to attach license and other 
metadata to publish your knowledge base.
The foaf:primaryTopic property could be used to link a kees:KnowledgeBaseDescription document to a kees:KnowledgeBase individual.

The **kees:Question** represents the *purpose* for the the knowledge base existence. In other words, the knoledge base exists to answer to *questions*. Question are natural language expressions that can be expressed as a query on a populated knowledge graph. The answer to a question results in tabular data, structured document, logic assertion or a translation of these in a natural language sentences.


The KEES vocabulary is expressed with OWL RDF in [kees.rdf file](v1/kees.rdf). The file was edited with Protégé editor.

Besides few classes and properties, KEES vocabulary defines some individuals:

- **kees:guard** a [SPARQL service description feature](https://www.w3.org/TR/sparql11-service-description/#sd-Feature) that states that the RDF store supports KEES guard specifications (see below)
 **kees:trustGraphMetric** defines a metric that allows to evaluate a trust rank for selected information in [daq framework](http://purl.org/eis/vocab/daq). 
- **kees:append** and **kees:replace** state two possible graph accrual policies: *append* policy affirms that,  if new facts found,
they are to be appended to existing data. The *replace* policy affirms that new data must replace all existing information.
- If you want to share/trade your knowledge base, simply attach your KEES plan, questions and  license to **kees:sharableKnowledge** object.
- **kees:namedGraphGenerator** is the role that a kees:Agent must use in building/updating a named graph.

## The KEES Language profile

The **KEES Language Profile** reuses some terms from existing vocabularies and adds mappings and restrictions to the KEES ontology.
In the rest of this document following  namespaces are used:

- dct: http://purl.org/dc/terms/
- dcterms: http://purl.org/dc/terms/
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

Dublin core vocabulary is used to annotate dataset. The properties dct:created, dct:modified are part of the KEES Language profile.

Knowledge base building activities MUST be traced using [PROV ontology](https://www.w3.org/TR/prov-overview/). KEES language profile
reuses prov:wasGeneratedBy, prov:wasInvalidatedBy, prov:qualifiedAssociation, prov:agent, prov:hadRole properties and prov:Role,
prov:Plan, prov:Activity classes.

Trustability is enabled by attaching quality observation to ingested graph. 
For this feature KEES language profile reuses the [Dataset Quality Vocabulary (daQ)](http://purl.org/eis/vocab/daq) with
qb:Observation class and daq:computedOn , daq:metric, daq:value and daq:isEstimated properties

The following picture sumarizes the main elements of the KEES language profile.

![uml](architecture/uml.png)


## RDF Store requirement

To Know the **provenance** of each statement, it is of paramount imortance to get an idea about data quality. For this reason, KEES requires that all statements must have a fourth element that links to a data source. This means that, for pratical concerns, the KEES knowledge base is a collection of quads, i.e. a triple plus a link to a metadata.

Any RDF Store that provides with a SPARQL endpoint and QUAD support is compliant with KEES. 

During knowledge base building and update the knowledge base could be in an inconsistent state.
If a statement with the subject <urn:kees:kb> and the predicate *dct:valid* exists, 
then it means that the Knowledge base is *safe* to be queried. Otherwhise queries the knowledge base should be considered *not safe*.

To declare that a RDF Store is ready to be safely queried, execute following SPARQL UPDATE statement:

```sparql
INSERT { <urn:kees:kb> dct:valid ?now }
WHERE { BIND( NOW() AS ?now) }
```

To declare that a RDF Store is *not safe*:

```sparql
DELETE {<urn:kees:kb> dct:valid ?x} WHERE { <urn:kees:kb> dct:valid ?x }
```

To check if a RDF Store is *safe*: 

```sparql
ASK { <urn:kees:kb> dct:valid [] }`
```

## SPARQL service requirements

A KEES compliant [SPARQL service](https://www.w3.org/TR/sparql11-service-description/) SHOULD expose the **kees:guard** feature. 
If a this feature is present,
the SPARQL endpoint MUST return the http *503 Error* when someone try to query a RDF Store that is not in the *safe* state.  A KEES compliant SPARQL endpoint SHOULD disable the guard feature if the http header "X-KEES-guard: disable" is present in the HTTP request.

A KEES compliant SPARQL service SHOULD alwais provide optimized http caching information [as described in Section 13 of RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html).


## KEES agent requirements

### Workflow

A KEES Agent SHOULD perform actions on a knowledge base on a logical sequence of four temporal phases called *windows*:

1. a startup  phase (**boot window**)  to initialize the knowledge base starting from one or more knowledge base descriptions
2. a time slot for the population of the Knowledge Base and to link data (**learning window**). It consists in the 
   execution of a plan that requires the downloading of at least an external resource.
3. a time slot for the data inference (**reasoning window**). It consists in the execution of a plan that requires only axioms and
   learned facts.
4. a time slot to access the Knowledge Base and to answering questions (**teaching window**)

The steps 2 and 3 can be iterated.

This sequence is called **KEES workflow** and it is a continuous integration process that starts on user request, scheduled time or 
after triggering an event (e.g. a dataset change).

### The Target graph

A target graph is a named graph in the knowledge base referenced by the property kees:build. In a KEES knowledge base,
every named graph shoud be referenced by exactly one plan through the kees:builds property.

Plans MUST provide to a KEES agent enough information to describe how to build or update the target graph.


### Plan pre-conditions

Plans MUST be evaluated only if all pre-conditions are satisfied. If just one pre-condition fails, the plan execution
MUST be skipped or postponed without changing the knowledge base.

There are two kinds of preconditions, related with two properties: kees:accualPeriodicity and kees:requires .


#### kees:requires pre-condition

The **kees:requires** range MUST be an URI that represents a resource in the knowledge base. Multiple kees:requires are allowed.

If no kees:requires is present, then the pre-condition is always satisfied and the rule MUST be executed.

If the target graph is not created, then the pre-condition is always satisfied and the rule MUST be executed.

If just one required URI is not present in the knowledge base, then the precondition is not satisfied and
the rule execution MUST be potsponed.

If all required resources exists and are older than the target graph creation date, then the precondition is  satisfied 
but the rule MUST be skipped. A KEES agent SHOLUD be able to use the dct:modified and dct:created properties to compare 
modification date.

If one required URI has not modification date, then the pre-condition is satisfied  and the rule MUST be executed.

A possible implementation on an algorithm that decides if all kees:requieres is satisfied :

```sparql
ASK { 
   VALUES ?plan { <here the uri of the plan> }
   {
      ?plan kees:requires ?uri; kees:builds ?graphName.
      ?graph sd:name ?graphName; dct:created ?graphCreationDate .
      OPTIONAL { ?uri dct:modified ?uriModificationDate }
      BIND( COALESCE(?uriModificationDate, NOW()) as ?lastModified)
      FILTER( ?lastModified > ?graphCreationDate )
      
   } UNION {
      FILTER NOT EXISTS { ?plan kees:requires [] }
   } UNION {
      FILTER NOT EXISTS {
      	?plan kees:builds ?graphName.
        ?graph sd:name ?graphName; dct:created ?graphCreationDate .
      }
   }
}
```


#### Accrual periodicity

kees:accualPeriodicity expects exactly a URI that describes an expected frequency (i.e. once a month, once a year) for updating a target graph.

The KEES agent SHOULD recognize at least all concepts in sdmx-code:freq scheme for dct:accrualPeriodicity and use these information to
decide if executing a plan or not. 

The accrual periodicity pre-condition is satisfied if one of this conditions matches:

- no kees:accualPeriodicity properties is defined 
- no target graph exists 
- the last modification date of the target graph plus the accrual frequency is greather of than current time

If accrual periodicity pre-condition is not satisfied then the rule MUST be skipped.


### Error conditions and error management

A KEES agent MUST update the RDF store safe statement when it enters or exits the teaching window. 

If a KEES agent was unable to complete succesfully a plan, it MUST abort if the target named graph was partially builded, otherwhise
it MUST annotate the named graph with the prov:InvalidatedAtTime property and continue.

If the KEES agent aborts its execution, the knowledge base MUST resulting in a "not safe" state.

In normal operations, the existence of prov:InvalidatedAtTime in a named graph MUST prevent the KEES agent to enter the teaching window.
A KEES agent COULD provide a way to force a safe state when some named graph contains a prov:InvalidatedAtTime property.

The existence of prov:InvalidatedAtTime in SHOULD be signaled by KEES agent. How to signal is implementation dependent

The existence of activities without a plan SHOULD be signaled by KEES agent. This condition does not prevent the 
KEES agent to enter the teaching window.

A KEES agent MUST abort in case of semantic inconsistences in *KEES knowledge base definition*.


### Graph constructors

A constructor is a resource referenced by the kees:from property that MUST provide enough information to a KEES agent to populate
the target named graph. A constructor can be a script in some language (i.e. SPARQL) or a data provider.

KEES does not impose any requirement for a constructor, but expects that a KEES agent SHOULD be smart enough to 
recognize and manage at least following kind of constructors:

- a **dereferenceable URL** that provides RDF triple using one of the standard RDF serialization. In this case
  the Agent SHOULD be able to download the resource content from the URL following HTTP(s) GET protocol specification (e.g. managing 
  redirection and  content negotiation) and to extract from it the information serialized according with one of the RDF standards:
  RDF/XML, turtle, json-ld, RDFa, Microdata, n3, N-Triples.
- an object of  type **sp:Construct** . In this case the KEES agent
  should be able to run a SPARQL query described in the object and injecting the results in the target graph specified in
  kees:builds property.
- an object of  type **sp:Update** . In this case the KEES agent
  should be able to execute the SPARQL Update script y described in the object  in the knowledge graph database. 
  The update script MUST NOT modify any  graph described in other plans.

SPARQL constructors should understand at least the sp:text property as defined in the
[SPIN W3C Member Submission 22 February 2011, updated 07 November, 2014](http://spinrdf.org/spin.html) .



For example, suppose that a KEES agent wants to execute this plan:

```turtle
:myplan kees:builds :mygraph kees:from <http://example.com/dataset.ttl> .
```

than it could run the following SPARQL update script:


```sparql
# prepare if something goes wrong
INSERT {[] sd:name :mygraph; prov:invalidatedAtTime ?now} WHERE {BIND(NOW() AS ?now)} ;

# Load resource in a temporary graph
LOAD <http://example.com/dataset.ttl> INTO GRAPH  <urn:tmp:graph>;

# REPLACE graph metadata in the default graph
DELETE { ?s ?p ?o }
INSERT {
   [] sd:name :mygraph;
      dct:creator  ;
      dct:created ?now;
      dct:modified ?now;
      prov:wasGeneratedBy [ a prov:Activity ;
         prov:used <http://example.com/dataset.ttl> ;
	 prov:qualifiedAssociation [ 
            prov:agent [ a prov:SoftwareAgent ] ;
	    prov:hadRole kees:namedGraphGenerator ;
	    prov:hadPlan ?plan 
	 ] ;     
      ]
} WHERE {
  ?g a sd:NamedGraph; sd:name :mygraph; (<>|!<>)* ?s . 
  ?s ?p ?o .
  
  ?plan kees:builds :mygraph.
  BIND( NOW() AS ?now )
};

# commit the transaction
MOVE SILENT GRAPH <urn:tmp:graph> TO :mygraph
```

In this case the KEES agent decided to store graph metadata in the default graph. Other agents could choice to store
graph metadata  in a separate named graph.

### Plan post-conditions

After a plan execution a KEES agent must evaluate the condition referred by the kees:assert property. Multiple kees:assert 
properties can be defined.

If just a post-condition fails, the KEES agent must invalidate the generated/updated graph and abort. KEES does not
specify any pre-ondition test order (in principle they can be evaluated in parallel)

Post conditions are always satisfied if no kees:assert property is present.

Post conditions are always not satisfied if the target graph was not updated for some reason.

A KEES agent sholud be able to evaluate post-condition with at least two methods:

- a post condition is satisfied if the range of the kees:assert property is the name of a graph defined in a plan and it exists
- a post condition is satisfied if the range of the kees:assert is a sp:Ask that evaluated to true



## KEES Axioms


### Implicit declaration of kees:sharedKnowledge 

The individual kees:sharedKnowledge  is alwas defined as 

```sparql
CONSTRUCT {kees:sharedKnowledge a kees:KnowledgeBase}
WHERE { FILTER NOT EXISTS {  kees:sharedKnowledge a kees:KnowledgeBase }}
```

### Functional property management

A KEES agent MUST be able to infer types from kees ontology functional properties.

```sparql
CONSTRUCT { ?s a ?c}
WHERE {
      ?s ?p ?o.
      ?p a owl:FunctionalProperty; rdfs:domain ?c.
      FILTER NOT EXISTS { ?s a ?c }
}
```


### A plan is alwais attached to a knowledge base

If a stand alone plan exists, it must be considered part of the kees:sharedKnowledge. 

```sparql
CONSTRUCT { kees:sharedKnowledge kees:hasPlan ?plan }
WHERE {?plan a kees:Plan FILTER NOT EXISTS {  ?x kees:hasPlan ?plan }}
```

### A question is alwais attached to a knowledge base

If a stand alone question exists, it must be considered attached to the kees:sharedKnowledge. 

```sparql
CONSTRUCT { kees:sharedKnowledge kees:answers ?question }
WHERE {?question a kees:Question FILTER NOT EXISTS {  ?x kees:answers ?question }}
```


### Inferred kees:from 

If exists a plan  without kees:from property, a default MUST be provided; e.g.:

```sparql
CONSTRUCT { ?plan kees:from ?graphUri } 
WHERE {
   ?plan kees:builds ?graphUri .
   FILTER NOT EXISTS { ?plan  kees:from [] }
}
```

### Inferred kees:accrualPeriodicity

If no *kees:accrualPolicy* exists, *kees:replace* MUST be used:

```sparql
CONSTRUCT { ?plan kees:accrualPolicy kees:replace} 
WHERE {
   FILTER NOT EXISTS { ?plan kees:accrualPolicy [] }
}
```

Exactly one *kees:accrualPolicy* must be present.


### Default data quality


If no explicit data quality observation records are present in the knowledge base, this axiom SHOULD applies:

```sparql
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


### Example

Suppose that there is knowledge base description file compsed by just one line:

```turtle
:myplan kees:builds <http://example.com/dataset.ttl>.
```

A KEES agent MUST be considered it equivalent to:

```turtle
<> a kees:KnowledgeBaseDescription;
   foaf:primaryTopic kees:sharedKnowledge .
   
kees:sharedKnowledge a kees:KnowledgeBase;
   kees:hasPlan :myplan .
:myplan a kees:Plan ;
   kees:builds <http://example.com/dataset.ttl> ;
   kees:from <http://example.com/dataset.ttl> ;
   kees:accrualPolicy kees:replace 
.
```

A smarter KEES agent SHOULD consider also:
```turtle
:myplan ;
   kees:requires <http://example.com/dataset.ttl> ;
   # Use  a minimal accualPeriodiciy to reduce DOS attacks
   kees:accualPeriodiciy sdmx-code:freq-m ; # a minute
   kees:asserts [ a sp:Ask; sp:text """ASK {
     ?x sd:name <http://example.com/dataset.ttl>; dct:created ?created; dct:modified ?modified.
     FILTER( ?modified >= ?created )
     FILTER NOT EXISTS{ ?x prov:InvalidatedAtTime []}}
   }"""]
.
```


## Integrity tests

Following axioms MUST always eval to true after the execution of the KEES axioms. If not KEES agent should abort.

### Cardinality = 1 

All OWL restrictions in KEES ontology that require a cardinality 1 MUST be satisfied.


### Cardinality = some

All OWL "sameof" restrictions in KEES ontology  MUST be satisfied.


## KEES agent protocol

A KEES Agent SHOULD accept as input one or more URL dereferencing to kees:KnowledgeBaseDescription resources
A KEES Agent SHOULD accept as input at least one kees:KnowledgeBaseDescription document

The input method is implentation dependent: a KEES agent can be implemented as a web service or as a command or as a job.

A KEES agent SHOULD accept, as optional parameter, a namespace to be used when it needs to create new resources.

KEES does not impose a way to detect if an agent is running or is aborted. 
But you MUST can always able to check if the knowledge base is in a safe state or not

KEES agent implementation SHOULD add some log and monitor features.

A KEES agent is expected to implement a process like:

1. Put the knowledge base in a *not safe* status.
2. Execute KEES axioms.
3. Ensure the integrity of the knowledge base descriptions. Abort if errors;
4. Find a plan that matches all pre-conditions, then take the appropriate action using the constructor and post-conditions.
5. Repeat steps 2-3-4 until it finds a matching plan.
6. Check if there are pending plan (i.e. plans with unsatisfied pre-condition). If yes abort.
7. Check that no named graph was invalidated.  If yes abort.
8. Enter teaching window and put the knowledge base in a *safe* status
9. (Optional) Print an execution report summary
10. (Optional) Provide execution log


## Example of a KEES knowledge Base Desctiption document

[** WARNING: THIS SECTION IS INFORMATIVE AND SUBJECTED TO MAYOR CHANGS **]

NOTE: all namespace declarations omitted to improve readability.

### Publishing knowledge base description as linked data resource

Create a file kees.ttl fit following content.

```turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix dct: <http://purl.org/dc/terms/> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .      
@prefix kees: <http://linkeddata.center/kees/v1#> .

<> a kees:KnowledgeBaseDescription; 
	dct:title "This dummy knowledge base is about teenagers in Europe"@en;
	dct:creator <https://example.org/profile/card#me>;
	dct:license <http://www.opendatacommons.org/odc-public-domain-dedication-and-licence/> .
	foaf:primaryTopic kees:sharableKnowledge
.

```


### Defining plans

Add to kees.ttl file:

```turtle
:a_learning_plan kees:builds :some_facts; kees:from <http://data.example.com/peoples_from_europe.rdf> .
:a_reasoning_plan dct:title "Compute if a people shoud to be considered a teenager"@en;
	kees:builds :inferences ;
	kees:requires :some_facts; 	
	kees:from [ a sp:Construct; sp:text """
		CONSTRUCT { ?person a ex:Teenager>  }
		WHERE {
			?person ex:birthDate ?birth .
			BIND( NOW() - ?birth AS ?age)
			FILTER (?age > 11 && ?age < 20 )
		}
	"""]	
.
	
```

### Attaching questions


Add to kees.ttl file:

```turtle
res_:teaching a kees:Question
	dct:title "Teenagers in europe"@en;
	kees:answerMethod [ a sp:Select; sp:text "SELECT (COUNT(DISTINT ?person) AS ?teenagers) WHERE {?person a ex:Teenager}"]
 ].
```

### adding trust info


Trust in dataset can be expessed with:

```turtle
res_:data_quality a qb:Observation ;
    daq:computedOn :some_facts ; 
    daq:metric kees:trustGraphMetric;
    daq:value 0.78 ;
    daq:isEstimated true .

res_:reasoning_quality a qb:Observation ;
    daq:computedOn :inferences ; 
    daq:metric kees:trustGraphMetric;
    daq:value 1.00 ;
    daq:isEstimated false .
```

## Contributing to this site

A great way to contribute to the site is to create an [issue](https://github.com/linkeddatacenter/kees/issues) on GitHub when you encounter a problem or something. We always appreciate it. You can also edit the code by yourself and create a pull request.


[Open-world assumption]: https://en.wikipedia.org/w/index.php?title=Open-world_assumption&oldid=871019791

All stuff here in the Creative Common (unless otherwise noted)


[RDF]: https://www.w3.org/TR/rdf11-primer/
