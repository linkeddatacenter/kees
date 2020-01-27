Implementing  KEES 
=======================================

**WARNING: WORKING IN PROGRESS**

Please before continue, have a look to the [KEES definitions and principles](README.md)

This working draft collects a proposal for a concrete KEES implementation.


## KEES Vocabulary

The **KEES vocabulary** defines few new terms in the  http://linkeddata.center/kees/v1#  namespace ( usual prefix *kees:*). 
It consists of some OWL classes and properties, mainly derived from existing ontologies. 

![uml](v1/images/uml.png)

A **kees:KnowledgeBase** is defined as a subclass of a [sd:Dataset](https://www.w3.org/TR/sparql11-service-description/#sd-Dataset) that, in turn, is a specialization of a dataset as described in the [VoID](https://www.w3.org/TR/void/). A kees:KnowledgeBase is
a collection of [sd:namedGraph](https://www.w3.org/TR/sparql11-service-description/#sd-NamedGraph) that contain the linked data.

The main class introduced by KEES vocabulary is the **kees:Plan** that describes how to create and update a named graph in the knowledge base, extracting facts extracted from a data source or derived from axioms. 

A **kees:KnowledgeBaseDescription** is a document that contains the description of the knowledge base with the purpose of
publishing and transferring  knowledge base building information.
It is a subclass of a [foaf:Document](http://xmlns.com/foaf/spec/#term_Document) that allows attaching license and other 
metadata to publish your knowledge base.
The foaf:primaryTopic property could be used to link a kees:KnowledgeBaseDescription document to a kees:KnowledgeBase individual.

The **kees:Question** represents the *purpose* for the knowledge base existence. In other words, the knowledge base exists to answer *questions*. Questions are natural language expressions that can be expressed as a query on a populated knowledge graph. The answer to a question results in tabular data, structured document, logic assertion or a translation of these in natural language sentences.

The **kees:Anomaly** is an annotation (oa:Annotation) thah mark a potential inconsistence of data respect language profile. 


The KEES vocabulary is expressed with OWL RDF in [kees.rdf file](v1/kees.rdf). The file was edited with Protégé editor.

Besides a few classes and properties, KEES vocabulary defines some individuals:

- **kees:guard** a [SPARQL service description feature](https://www.w3.org/TR/sparql11-service-description/#sd-Feature) that states that the RDF store supports KEES guard specifications (see below)
 **kees:trustGraphMetric** defines a metric that allows to evaluate a trust rank for selected information in [daq framework](http://purl.org/eis/vocab/daq). 
- **kees:append** and **kees:replace** state two possible graph accrual policies: *append* policy affirms that,  if new facts found,
they are to be appended to existing data. The *replace* policy affirms that new data must replace all existing information.
- If you want to share/trade your knowledge base, simply attach your KEES plan, questions and license to **kees:sharableKnowledge** object.
- **kees:namedGraphGenerator** is the role that a kees:Agent must use in building/updating a named graph.

## The KEES Language profile

The **KEES Language Profile** reuses some terms from existing vocabularies and adds mappings and restrictions to the KEES ontology.
In the rest of this document following  namespaces are used:

- dct: http://purl.org/dc/terms/
- dcterms: http://purl.org/dc/terms/
- sdmx-code: http://purl.org/linked-data/sdmx/2009/code
- sd: http://www.w3.org/ns/sparql-service-description
- foaf: http://xmlns.com/foaf/0.1/
- prov: http://www.w3.org/ns/prov#
- ao: http://www.w3.org/ns/oa#
- void: http://rdfs.org/ns/void
- vann: http://purl.org/vocab/vann/
- voaf: http://purl.org/vocommons/voaf#
- spin:  http://spinrdf.org/spin
- rdf:  http://www.w3.org/1999/02/22-rdf-syntax-ns
- rdfs:    http://www.w3.org/2000/01/rdf-schema#
- owl: http://www.w3.org/2002/07/owl#
- kees: http://linkeddata.center/kees/v1#

Dublin core vocabulary is used to annotate dataset. The properties dct:created, dct:modified are part of the KEES Language profile.

Knowledgebase building activities MUST be traced using [PROV ontology](https://www.w3.org/TR/prov-overview/). KEES language profile
reuses prov:wasGeneratedBy, prov:wasInvalidatedBy, prov:qualifiedAssociation, prov:agent, prov:hadRole properties and prov:Role,
prov:Plan, prov:Activity classes.

Trustability is enabled by attaching quality observation to the ingested graph. 
For this feature KEES language profile reuses the [Dataset Quality Vocabulary (daQ)](http://purl.org/eis/vocab/daq) with
qb:Observation class and daq:computedOn , daq:metric, daq:value and daq:isEstimated properties

The annotation use [WEB Annotation](http://www.w3.org/ns/oa) vocabulary.

The following picture summarizes the main elements of the KEES language profile.

![uml](architecture/uml.png)


## RDF Store requirement

To Know the **provenance** of each statement, it is of paramount importance to get an idea about data quality. For this reason, KEES requires that all statements must have a fourth element that links to a data source. This means that, for practical concerns, the KEES knowledge base is a collection of quads, i.e. a triple plus a link to metadata.

Any RDF Store that provides with a SPARQL endpoint and QUAD support is compliant with KEES. 

During knowledge base building and update the knowledge base could be in an inconsistent state.
If a statement with the subject <urn:kees:kb> and the predicate *dct:valid* exists, 
then it means that the Knowledge base is *safe* to be queried. Otherwise queries the knowledge base should be considered *not safe*.

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
If this feature is present,
the SPARQL endpoint MUST return the http *503 Error* when someone tries to query an RDF Store that is not in the *safe* state.  A KEES compliant SPARQL endpoint SHOULD disable the guard feature if the http header "X-KEES-guard: disable" is present in the HTTP request.

A KEES compliant SPARQL service SHOULD always provide optimized http caching information [as described in Section 13 of RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html).


## KEES agent requirements

### KEES Workflow

A KEES Agent SHOULD perform actions on a knowledge base on a logical sequence of four temporal phases called *windows*:

1. a startup  phase (**boot window**)  to initialize the knowledge base starting from one or more knowledge base descriptions
2. a time slot for the population of the Knowledge Base and to link data (**learning window**). It consists of the 
   execution of a plan that requires the downloading of at least an external resource.
3. a time slot for the data inference (**reasoning window**). It consists of the execution of a plan that requires only axioms and
   learned facts.
4. a time slot to access the Knowledge Base and to answering questions (**teaching window**)

Steps 2 and 3 can be iterated.

The plans depend on each other. Dependencies can be explicitly defined using the kees:requieres properties in plans or implicitly
defining an RDF list of plans referenced by the kees:planSequence in a kees:Knowledgbase resource. A knowledge base can enter the
teaching window only when all plans are fulfilled.

The sequence of plan execution is called **KEES workflow** and it is a continuous integration process that starts on a triggered event (e.g change in source data, a user request or a scheduled job) on the knowledge base that is in the teaching windows.

### The Target graph

A target graph is a named graph in the knowledge base referenced by the property kees:build. In a KEES knowledge base,
every named graph should be referenced by exactly one plan through the kees:builds property.

Plans MUST provide to a KEES agent enough information to describe how to build or update the target graph.


### Plan pre-conditions

Plans MUST be evaluated only if all pre-conditions are satisfied. If just one pre-condition fails, the plan execution
MUST be skipped or postponed without changing the knowledge base.

There are two kinds of preconditions, related to two properties: kees:accualPeriodicity and kees:requires .


#### kees:requires pre-condition

The **kees:requires** range MUST be a URI that represents a resource in the knowledge base. Multiple kees:requires are allowed.

If no kees:requires is present, then the pre-condition is always satisfied and the rule MUST be executed.

If the target graph is not created, then the pre-condition is always satisfied and the rule MUST be executed.

If just one required URI is not present in the knowledge base, then the precondition is not satisfied and
the rule execution MUST be postponed.

If all required resources exist and are older than the target graph creation date, then the precondition is satisfied
but the rule MUST be skipped. A KEES agent SHOULD be able to use the dct:modified and dct:created properties to compare modification date.

If one required URI has not modification date, then the pre-condition is satisfied and the rule MUST be executed.

A possible implementation on an algorithm that decides if all kees:requires is satisfied :

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

The KEES agent SHOULD recognize at least all concepts in sdmx-code:freq scheme for dct:accrualPeriodicity and use this information to
decide if executing a plan or not. 

The accrual periodicity pre-condition is satisfied if one of these conditions matches:

- no kees:accualPeriodicity properties is defined 
- no target graph exists 
- the last modification date of the target graph plus the accrual frequency is greater of than current time

If accrual periodicity pre-condition is not satisfied then the rule MUST be skipped.


### Error conditions and error management

A KEES agent MUST update the RDF store safe statement when it enters or exits the teaching window. 

If a KEES agent was unable to complete successfully a plan, it MUST abort if the target named graph was partially built, otherwise
it MUST annotate the named graph with the prov:InvalidatedAtTime property and continue.

If the KEES agent aborts its execution, the knowledge base MUST result in a "not safe" state.

In normal operations, the existence of prov:InvalidatedAtTime in a named graph MUST prevent the KEES agent to enter the teaching window.
A KEES agent COULD provide a way to force a safe state when some named graph contains a prov:InvalidatedAtTime property.

The existence of prov:InvalidatedAtTime in SHOULD be signaled by KEES agent. How to signal is implementation dependent

The existence of activities without a plan SHOULD be signaled by KEES agent. This condition does not prevent the 
KEES agent to enter the teaching window.

A KEES agent MUST abort in case of semantic inconsistencies in *KEES knowledge base definition*.


### Plan constructors

A constructor is a resource referenced by the kees:from property that MUST provide enough information to a KEES agent to populate
the knowledge base. A constructor can be a script in some language (i.e. SPARQL) or a data provider.

KEES does not impose any requirement for a constructor, but expects that a KEES agent SHOULD be smart enough to 
recognize and manage at least following kind of constructors:

- a **dereferenceable URL** that provides RDF triple using one of the standard RDF serializations. In this case
  the Agent SHOULD be able to download the resource content from the URL following HTTP(s) GET protocol specification (e.g. managing 
  redirection and  content negotiation) and to extract from it the information serialized according to one of the RDF standards:
  RDF/XML, turtle, json-ld, RDFa, Microdata, n3, N-Triples.
- an object of type **sp:Construct** . In this case the KEES agent
  should be able to run a SPARQL query described in the object and injecting the results in the target graph specified in
  kees:builds property.
- an object of type **sp:Update** . In this case the KEES agent
  should be able to execute the SPARQL Update script y described in the object in the knowledge graph database. 
  The update script MUST NOT modify any graph described in other plans.

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

In this case, the KEES agent decided to store graph metadata in the default graph. Other agents could choose to store
graph metadata in a separate named graph.


### Plan destructor

A destructor is a resource referenced by the kees:destructor property that MUST provide enough information to a KEES agent to remove
from the knowledge created by constructors.
A destructor can be a script in some language (i.e. SPARQL) 

KEES does not impose any requirement for a destructor but expects that a KEES agent SHOULD be smart enough to recognize and manage at least a resource of type **sp:Update** . In this case, the KEES agent
should be able to execute the SPARQL Update script described in the resource in the knowledge graph database. 
The update script MUST NOT destroy any graph described in other plans.

SPARQL destructors should understand at least the sp:text property as defined in the
[SPIN W3C Member Submission 22 February 2011, updated 07 November, 2014](http://spinrdf.org/spin.html) .

If no destructor is specified, as default behavior, all named graph referenced by kees:changes propery MUST be silently dropped.



### Plan post-conditions

After a plan execution a KEES agent must evaluate the condition referred by the kees:assert property. Multiple kees:assert 
properties can be defined.

If just a post-condition fails, the KEES agent must invalidate the generated/updated graph and abort. KEES does not
specify any pre-condition test order (in principle they can be evaluated in parallel)

Postconditions are always satisfied if no kees:assert property is present.

Postconditions are always not satisfied if the target graph was not updated for some reason.

A KEES agent should be able to evaluate post-condition with at least two methods:

- a postcondition is satisfied if the range of the kees:assert property is the name of a graph defined in a plan and it exists
- a postcondition is satisfied if the range of the kees:assert is an sp:Ask that evaluated to true



## KEES Axioms


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

If a stand-alone plan exists, it must be considered part of the kees:sharedKnowledge. 

```sparql
CONSTRUCT { kees:sharedKnowledge kees:hasPlan ?plan }
WHERE {?plan a kees:Plan FILTER NOT EXISTS {  ?x kees:hasPlan ?plan }}
```

### A question is always attached to a knowledge base

If a stand-alone question exists, it must be considered attached to the kees:sharedKnowledge. 

```sparql
CONSTRUCT { kees:sharedKnowledge kees:answers ?question }
WHERE {?question a kees:Question FILTER NOT EXISTS {  ?x kees:answers ?question }}
```


### Inferred constructor

If exists a plan without kees:from property, a default MUST be provided; e.g.:

```sparql
CONSTRUCT { ?plan kees:from ?graphUri } 
WHERE {
   ?plan kees:builds ?graphUri .
   FILTER NOT EXISTS { ?plan  kees:from [] }
}
```

### Inferred  plan dependencies

The kees:planSequence propery MUST refer an rdf:list that MUST be considered as a shortcut to specify plan dependencies. 

For example:

`ex:kb kees:workflow ( ex:plan1 ex:plan2)` means that  `ex:plan2 kees:requires ex:plan1`

This implies that all list element are kees:Plan

CONSTRUCT { ?plan2 kees:requires ?plan1 } 
WHERE {
   ?x rdf:first ?plan1; rdf:rest/rdf:first ?plan2.
   ?plan1 a kees:Plan.
   ?plan2 a kees:Plan
   FILTER NOT EXISTS { ?plan2 kees:requires ?plan1 }
}

### kees:changes always includes a constructor

The resources referred by kees:from MUST be referenced also by kees:changes


```sparql
CONSTRUCT { ?plan kees:changes ?resource } 
WHERE {
   ?plan kees:builds ?resource .
   FILTER NOT EXISTS { ?plan  kees:changes resource }
}
```


###  Constructor must have always a type

The object referred by the of kees:from propery, MUS have a type. If not provided rdfs:Resource MUST be used.

```sparql
CONSTRUCT { ?fromUri a rdfs:Resource } 
WHERE {
   ?plan kees:from ?fromUri .
   FILTER NOT EXISTS { ?fromUri a [] }
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


If no explicit data quality observation records are present in the knowledge base, this axiom SHOULD apply:

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

### Graph invalidation

If a terminated prov:activity invalidates a graph, then the graph is invalidated when the activity ended


```sparql
CONSTRUCT  {?graph prov:invalidatedAtTime ?endTime} 
WHERE {
      ?activiti a prov:Activity; prov:endedAtTime ?endTime; prov:invalidated ?name.
      ?graph sd:name ?name.
      FILTER NOT EXISTS { ?graph prov:invalidatedAtTime ?endTime }
}
```

### Example

Suppose that there is a knowledge base description file composed by these lines:

```turtle
kees:sharedKnowledge kees:planSequence( 
    [kees:builds <http://example.com/dataset.ttl>]
    [kees:builds ex:g1 kees:from <http://example.com/dataset2.ttl>]
) .
```

A KEES agent MUST be considered it equivalent to the configuration:

```turtle
<> a kees:KnowledgeBaseDescription;
   foaf:primaryTopic kees:sharedKnowledge .
   
kees:sharedKnowledge a kees:KnowledgeBase;
   kees:hasPlan _:p1 , _:p2.
   
_:p1  a kees:Plan ;
   kees:builds <http://example.com/dataset.ttl> ;
   kees:from <http://example.com/dataset.ttl> ;
   kees:changes <http://example.com/dataset.ttl> ;
   kees:requires <http://example.com/dataset.ttl> ;
   kees:accualPeriodiciy sdmx-code:freq-m ; # minimum a minute to avoid DOS
   kees:accrualPolicy kees:replace 
.

_:p2  a kees:Plan ;
   kees:builds ex:g1 ;
   kees:from <http://example.com/dataset2.ttl> ;
   kees:changes ex:g1 ;
   kees:requires <http://example.com/dataset2.ttl>, <http://example.com/dataset.ttl> ;
   kees:accualPeriodiciy sdmx-code:freq-m ; # minimum a minute to avoid DOS
   kees:accrualPolicy kees:replace 
.
```

Supposing that a KEES Agent runs on 06/11/2019, it could produce following contents in the knowledge base:

```turtle
<urn:agent:config>: {
    #... same as the previous configuration
    
    ex:theAgent a foaf:Agent, prov:SoftwareAgent; foaf:name "A smart KEES agent".
}

<http://example.com/dataset.ttl> : {
    _:g1 a sd:NamedGraph; sd:name <http://example.com/dataset.ttl>
        dct:created "2019-11-06"^^xsd:date;
        dct:modified "2019-11-06"^^xsd:date;
        prov:wasGeneratedBy [ a prov:Activity ;
            prov:used 
                <http://example.com/dataset.ttl>;
            prov:qualifiedAssociation [
                prov:agent ex:theAgent;
                prov:role kees:namedGraphGenerator;
                prov:plan _:p1
            ]
        ]
    .
    
    #... here the data extracted from http://example.com/dataset.ttl
    
}

ex:g1 : {
    _:g2  a sd:NamedGraph sd:name ex:g1
        dct:created "2019-11-06"^^xsd:date;
        dct:modified "2019-11-06"^^xsd:date;
        prov:wasGeneratedBy [ a prov:Activity ;        
               kees:used 
                <http://example.com/dataset2.ttl>, 
                <http://example.com/dataset.ttl> ;
            prov:qualifiedAssociation [
                prov:agent ex:theAgent;;
                prov:role kees:namedGraphGenerator;
                prov:plan _:p2
            ]
        ]
        
        #... here the data extracted from http://example.com/dataset2.ttl
    .
}

```


## Integrity tests

Following axioms MUST always eval to true after the execution of the KEES axioms. If not KEES agent should abort.

### Cardinality = 1 

All OWL restrictions in KEES ontology that requires a cardinality 1 MUST be satisfied.


### Cardinality = some

All OWL "same of" cardinality restrictions in KEES ontology  MUST be satisfied.


### No circular references exists

Plan dependency can determine  circulal referenca (i.e. planA requires planB and planB requires plan A.

Such circular references SHOULD be early detected by the KEES agent.


## KEES agent protocol

A KEES Agent MUST accept as input the URI of a KnowledgeBase to be processed. If none provided, kees:sharedKnowledge MUST be used.

A KEES Agent SHOULD accept as input one or more URL dereferencing to kees:KnowledgeBaseDescription resources.

A KEES Agent SHOULD accept as input at least one kees:KnowledgeBaseDescription document.

The input method is implementation dependent: a KEES agent can be implemented as a web service or as a command or as a job.

A KEES agent SHOULD accept, as an optional parameter, a namespace to be used when it needs to create new resources.

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


## Example of a KEES knowledge Base Description document

[** WARNING: THIS SECTION IS INFORMATIVE AND SUBJECTED TO MAYOR CHANGS **]

NOTE: all namespace declarations omitted to improve readability.

### Publishing knowledge base description as a linked data resource

Create a file kees.ttl fit the following content.

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


Trust in the dataset can be expressed with:

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
