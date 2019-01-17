KEES: Knowledge Exchange Engine Service 
=======================================

**WARNING: WORKING IN PROGRESS**

In order to let computers to work for us, they must understand data: 
not just the grammar and the syntax but the real meaning of things. 

KEES proposes some specifications to add metadata to a *domain knowledge* in order to make it tradeable and shareable. 

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

The **Question** represents the reason for the the knowledge base existence. In other words, the knoledge base exists to answer to *questions*. Question are natural language expressions that can be expressed as parametric SPARQL queries on a populated knowledge graph. The answer to a question can be a table of data, a structured document, a boolean or a translation of these in a natural language sentences.

**Trust** is another key concept in KEES. The [Open-world assumption] and RDF allow to mix any kind of information, even when information that are incoerent. For instance, suppose that your TBOX defines a property "person:hasMom" that require a cardinality of 1 (i.e. every person has just one "mom"), your knowledge base could contains two different fact (:jack person:hasMom :Mary) and (:jack person:hasMom :Giulia), in order to take decision about who is jack's mom you need trust in your data. If you are sure about the veridicicy of all data in the knowledge base, you can deduct that :Mary and :Giulia are two names for the same person. If you are not so sure, you have two possibility: choose the most trusted statement with respect some criteria (even casually if both statemenst have the same trust rank) or to change the TBOX , allowing a person to have more than one mom. In any case you need to get an idea about _your_ trust on each statement (both ABox and Tbox) in the knowlege base. At least you want to know the **provenance** and all metadata of all information in your knowledge base because the trust on a single data often derives from the trust of its source or in the creator of the data source.

The **Language Profile** (or **Application profile**) is the portion of the vocabularies (TBOX) that describe the knowledge that are recognized by a specific software application. The language profile contains **domain specific axioms** and is normally contained in the Tbox partition of a knowledge base. 

An **axiom** describes how to generate/validate knowledge base statemensts using entailment inferred by language profile semantic.For example an axiom can be described with OWL and evaluated by a OWL reasoner or described with SPARQL QUERY construct or with SPARQL UPDATE statements.

A **rule** express a logic deduction in the form of if *some facts exists* then *new facts genetarted*. Like axioms ,rules can be 
described with SPARQL QUERY construct or with SPARQL UPDATE statements.

KEES allows to define **test conditions** that can be realized with an ASK SPARQL operation. 

KEES does not restrict the specification for axioms, rules nor for tes condition representation to SPARQL. A KEES agent can recognize additional languages.


## KEES Specification

## KEES Vocabulary

The **KEES vocabulary** defines few new terms the  http://linkeddata.center/kees/v1#  namespace ( usual prefix *kees:*). 
It consists of some OWL classes and properties, mainly derived from existing ontologies. 

All knowledge base building activities MUST be traced using PROV ontology. KEES require that every build activity must be associated with a plan.

The main classes introduced by KEES vocabulary are **kees:IngestionPlan** and **kees:ReasoningPlan**; both that are 
specialization of prov:Plan class. The first describes how to create a graph in the knowledge base extracting facts from a data source, 
the second describes how to materialize new information from existing knowledge base facts.

KEES vocabulary is expressed with OWL RDF in [kees.rdf file](v1/kees.rdf). The file was edited wit Protégé editor.

Besides few classes and properties, KEES vocabulary defines some individuals:

- **kees:guard** a [SPARQL service description](https://www.w3.org/TR/sparql11-service-description/#sd-Feature) feature that states that the RDF store supports KEES guard specifications (see below)
- **kees:trustStatementMetric** defines a  trust metric computed on specific rdf statement.
 **kees:trustPropertytMetric** defines a  trust metric computed on specific property, 
 **kees:trustSubjectMetric** defines a  trust metric computed on specific subject type,
 **kees:trustGraphMetric** defines a metric that evaluate a trust value for a graph with a specific name. 
   All can be used in graph quality observations.
- **kees:update** and **kees:create** state two possible graph accrual policies

KEES vocabulary defines some datatypes:

- **kees:sparqlQueryConstructOperation** states the datatype of a literal string containing a sparql query CONSTRUCT operation. 
- **kees:sparqlQuerySelectOperation** states the datatype of a literal string containing a sparql query SELECT operation.
- **kees:sparqlQueryDescribeOperation** states the datatype of a literal string containing a sparql query DESCRIBE operation.
- **kees:sparqlQueryAskOperation** states the datatype of a literal string containing a sparql query ASK operation.
- **kees:sparqlUpdateScript** states the datatype of a literal string containing a sparql update scrirpt.

The **KEES Language Profile** reuses following vocabularies:

- dct: http://purl.org/dc/terms/
- qb: http://purl.org/linked-data/cube#
- sdmx-code: http://purl.org/linked-data/sdmx/2009/code#
- daq: http://purl.org/eis/vocab/daq# 
- sd: http://www.w3.org/ns/sparql-service-description#
- prov: http://www.w3.org/ns/prov#
- kees: http://linkeddata.center/kees/v1#


The following picture sumarize the KEES language profile.

![uml](architecture/uml.png)


TODO: KEES language profile restrictions is formally expressed in [SHACL constraints file](v1/kees-profile.rdf)

A **KEES compliant application** is a Semantic Web Application that is not in conflict with the KEES Language Profile. 

A **KEES Agent** is a software process that understands a portion the KEES language profile and that it is able to do actions on a 
knowledge base. For instance, it could be able to ingest data and/or to answer some questions.

At the end, **KEES configuration** is a dataset describing a knowledge base. A KEES Agent should be able to rebuild the whole knowlege graph
just looking to the KEES configuration. Because different KEES configurations can be safely merged in a single new configuration, and this make knowledge domains shareable. Because a dataset can be protected with a license, you can sell your knowledge base (that is different to sell the data contained in the  knowlede about the data) making knowledge tradeable. 

The KEES configuration can be included in the knowledge base in the graph named <urn:kees:configuration> or kept as separate resource.


## RDF Store requirement

Knowing the statement **provenance** is the most usefull way to get an idea about its trustability. For this reason, KEES requires that any statement must have a fourth element that links to metadata that describe any statement in the knowledge base. This means that, for pratical concerns, the KEES knowledge base is a collection of quads, i.e. a triple plus a link to a metadata.

Any RDF Store that provides with a SPARQL endpoint and QUAD support is compliant with KEES. Following requirement applies:

- If a statement with subject <urn:kees:kb> and predicate dct:valid is present in the default graph, this  means that the Knowledge base is *safe* to be queried. Otherwhise the knowledge base should be considered *not safe*.

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
the endpoint MUST return 503 Error of any SPARQL QUERY that happens on a RDF Store that is not in the  *safe* state.  A KEES compliant sparql endpoint SHOULD MUST this feature if the http header "X-KEES-guard: disable" is present.

A KEES compliant sparql service MUST expose the  **kees:workflow** feature. The workflow plans must be attached to the defatul dataset of the service.

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

### Error conditions and error management

A KEES agent MUST update the RDF store safe statement when it enters or exits the teaching window. 

If the KEES agent abort execution, the knowledge base must be marked as "not safe", thus the knowledge base MAY be inconsistent.

The existence of prov:InvalidatedAtTime in a named graph MUST prevent the KEES agent to enter the teaching window.

The existence of prov:InvalidatedAtTime in SHOULD be signaled by KEES agent. How to signal is implementation dependent

The existence of activities without a plan SHOULD be signaled by KEES agent. This condition does not prevent the 
KEES agent to enter the teaching window.

A KEES agent MUST abort if it was unable to find a way to build a graph. 

A KEES agent must abort in case of semantic inconsistences in kees knowledge base definition. KEES does not impose
restriction on knowledge base consistence.

If a KEES agent was unable to complete succesfully a plan, it MUST abort if the target named graph was partially builded, otherwhise
it MUST annotate the named graph with the prov:InvalidatedAtTime property and continue.

## Ingestion plans behaviour and axioms

If  kees:dataSource attribute is not present a KEES agent MUST reuse  kees:builds value, e.g.:

```
CONSTRUCT { ?plan kees:dataSource ?g } 
WHERE {
   ?plan kees:builds ?g
   FILTER NOT EXISTS { ?plan kees:dataSource ?g }
}
```

The KEES agent SHOULD ignore an ingestion plan if the named graph exists and it is newer than kees:dataSource. 
How to evaluate this condition is implementation dependent but a data source without a clear modification date MUST 
be considered always newer than any existing named graph.

The KEES agent SHOULD recognize all concepts in sdmx-code:freq scheme for dct:accrualPeriodicity and use these information to
decide if executing a plan or not. How to manage dct:accrualPeriodicity depends from KEES Agent implementation.

## Reasoning plans behaviour and axioms

If  kees:builds attribute is not present a KEES agent MUST reuse  generate a new unique graph name, e.g.:

```
CONSTRUCT { ?plan kees:builds ?g } 
WHERE {
   BIND( UUID() as ?g ? }
   FILTER NOT EXISTS { ?plan kees:dataSource ?g }
}
```

The KEES agent SHOULD execute an reasoning plan only if all required xonditions eval to "true". 

 
## conditional properties evaluation in plan execution

The KEES agent MUST be able to recognize  kees:assert and kees:requires properties as kees:sparqlQueryAskOperation string, or as
a name of a graph.

If is a string the condition is true if the SPARQL ASK operation construct returns "true". An URI means that the 
named graph exists and it is updated.

The KEES agent MUST abort if kees:assert is present and evaluates to anything different from "true".


### KEES agent protocol

A KEES Agent MUST be able to accept in input one or more URL dereferencing to kees:KnowledgeBaseDescription resources.
The input method is implentation dependent. A KEES agent can be implemented as a web service or as a command or as a job.

There is no direct way to detect if an agent is running or is aborted. But you can always check if the knowledge base is in a
safe state. 

KEES agent implementation SHOULD add some log and monitor features.

KEES agent should be able to infer types from functional properties.

### Accrual policies

A KEES agent MUST recognize *kees:modify* and *kees:create* individuals as valid object for dct:accrualPolicy property in a kees:Plan. In case of inconsistencies or if no dct:accrualPolicy property is specified, the agent MUST choose kees:create.

If the accrual policy is kees:create, the named graph and all related metadata MUST be deleted and recreated BEFORE
to execute accrual policies.

### Accrual methods

A KEES agent should try to load all required resources specified by dct:requires property, in the named graph specified in
kees:builds property taking into consideration the resource types and accual methods.

At least, the agent is expected to execute perform like the SPARQL UPDATE statement "LOAD <url> INTO <graph>" construct.
	
If the dct:accrualMethod is a string with datatype *kees:sparqlQueryConstructOperation* then the agent MUST evaluate it
and add the constructed triples added to the named graph pointed by the plan

If the dct:accrualMethod is a string with datatype *kees:sparqlUpdate* then then the agent MUST evaluate it.


## Examples

[** WARNING: THIS SECTION IS INFORMATIVE AND SUBJECTED TO MAYOR CHANGS **]


## a knowledge base description dectaration

```
<> a kees:KnowledgeBaseDescription; foaf:primaryTopic kees:sharableKnowledge.
```

## Defining a workflow

```
kees:sharableKnowledge kees:workflow (
	[ 
		kees:builds <http://data.example.com/peoples_from_europe.rdf> ;
		dct:accrualPeriodicity sdmx-code:freq-Y
	]		
	[ 
		dct:title "Compute if a people is to be considered a teenager";
		kees:builds <urn:graph:inference1> ;
		kees:onlyIf """
			ASK {
				GRAPH ?g { 
					sd:name <http://data.example.com/peoples_from_europe.rdf> ;
					dct:modified ?dataSourceUpdate;
				}
				GRAPH ?me {
					sd:name <urn:graph:inference1> ;
					dct:modified ?inferenceUpdate;
				}
				FILTER(  ?dataSourceUpdate > ?inferenceUpdate )
			}
		"""^^kees:sparqlQueryAskOperation; 	
		dct:accrualMethod """
			PREFIX dbo: <http://dbpedia.org/ontology/>
			CONSTRUCT { ?person a <urn:class:teenager>  }
			WHERE {
				?person dbo:birthDate ?birth .
				BIND( NOW() - ?birth AS ?age)
				FILTER (?age > 11 && ?age < 20 )
			}
		"""^^kees:sparqlQueryConstructOperation 	
	]
) .
	
```

### Attaching questions

```
kees:sharableKnowledge kees:answers [  a kees:Question
	dct:title "Teenagers born in Berlin"@en;
	kees:answeredBy """
		PREFIX dbo: <http://dbpedia.org/ontology/>
		PREFIX : <http://dbpedia.org/resource/>

		SELECT DISTINCT ?person  
		WHERE { 
			?person a <urn:class:teenager>;
				dbo:birthPlace :Berlin. 
		}
	"""^^kees:sparqlQuerySelectOperation 
 ].
```

### adding trust info


Trust in a specific statement can be expessed with:
```
[] a qb:Observation ;
    daq:computedOn ( kees:anySubject, dbo:birthPlace, kees:anyObject) ; 
    daq:metric kees:trustMetric ;
    daq:value 0.10 ;
    daq:isEstimated true .
```

or trust in dataset can be expessed with:

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


### simple reasoning

This states that a some Infereces SHOULD created if the knowledge base is not empty 

```
[] a kees:Reasoning ;
	dct:accrualPolicy "ASK {?s ?p ?o}"^^kees:sparqlQueryAskOperation;
	kees:accrualMethod (
		"Drop all triples in inferred named graph"@en 
		"Evaluate base skos axioms"@en
	)
.

```

Note that in this case a KEES Agent that the able to understand the accrual method text is required to perform these inferences.

## Contributing to the site

A great way to contribute to the site is to create an [issue](https://github.com/linkeddatacenter/kees/issues) on GitHub when you encounter a problem or something. We always appreciate it. You can also edit the code by yourself and create a pull request.


[Open-world assumption]: https://en.wikipedia.org/w/index.php?title=Open-world_assumption&oldid=871019791

All stuff here in the Creative Common (unless otherwise noted)


[RDF]: https://www.w3.org/TR/rdf11-primer/
