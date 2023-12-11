Implementing KEES 
=======================================

Please before continue, have a look to the [KEES definitions and principles](README.md)

This working draft collects a proposal for a concrete KEES implementation.


## KEES Vocabulary

The **KEES vocabulary** defines few new terms in the  http://linkeddata.center/kees/v1#  namespace ( usual prefix *kees:*). 
It consists of few terms, mainly derived from existing ontologies: 

![uml](v1/images/uml.png)


"`kees:KnowledgeGraph` is a subclass of [sd:Service](https://www.w3.org/TR/sparql11-service-description/#sd-Service), indicating that the annotated subject represents a service compliant with the KEES protocol. It can offer a variety of optional properties, empowering a KEES-compliant agent to signal the completion of a window within a KEES cycle. `kees:KnowledgeGraph` represents a *singleton*; for each Graph Store, only one `sd:endpoint` can be associated with an instance of this class.

`kees:Activity`
: is a subclass of [prov:Activity](https://www.w3.org/TR/prov-o/#Activity) that annotates an action performed by a KEES compliant agent


`kees:Boot`, `kees:Ingestion`, `kees:Reasoning`, `kees:Enriching`, `kees:Publishing`, `kees:Versioning`
: are subclasses of [prov:Activity](https://www.w3.org/TR/prov-o/#Activity) related to specific KEES cycle windows activities


Besides classes, KEES vocabulary defines some individuals:

`kees:trustLevel`
: it is a predefined  instance of [dqv:Metric](https://www.w3.org/TR/vocab-dqv/#dqv:Metric) to be used in a [dqv:QualityMeasurement](https://www.w3.org/TR/vocab-dqv/#dqv:QualityMeasurement) to define a trust level in a [dqv:UserQualityFeedback](https://www.w3.org/TR/vocab-dqv/#dqv:UserQualityFeedback)  according with the [Data Quality Vocabulary](https://www.w3.org/TR/vocab-dqv/). The `dqv:value` property is expected to be a xsd:decimal in the range 0-1

`kees:append` and `kees:replace`
: state two instances of [dct:AccrualPolicy](https://www.dublincore.org/specifications/dublin-core/collection-description/accrual-policy/): *append* policy affirms that,  if new facts found,
they are to be appended to existing data. The *replace* policy affirms that new data must replace all existing information.


`kees:validity`
: it is a predefined instance of [sd:Feature](https://www.w3.org/TR/sparql11-service-description/#sd-Feature) that indicates that the annotated service supports the ability to detect if the knowledge graph is stable

`kees:locking`
: it is a predefined instance of [sd:Feature](https://www.w3.org/TR/sparql11-service-description/#sd-Feature) that indicates that the annotated service supports locking protocol



The whole KEES vocabulary is expressed with OWL RDF and available in [kees.rdf file](v1/kees.rdf).


## The KEES Language profile

The **KEES Language Profile** reuses some terms from existing vocabularies and adds mappings and restrictions to the KEES ontology.
In the rest of this document following  namespaces are used:

Absolutely! Here's the Markdown table with the first column ordered alphabetically:

| Prefix   | URL                                              |
|----------|--------------------------------------------------|
| dct      | http://purl.org/dc/terms/                         |
| dqv      | http://www.w3.org/ns/dqv#                         |
| dcterms  | http://purl.org/dc/terms/                         |
| kees     | http://linkeddata.center/kees/v1#                 |
| oa       | http://www.w3.org/ns/oa#                          |
| owl      | http://www.w3.org/2002/07/owl#                    |
| prov     | http://www.w3.org/ns/prov#                        |
| dqv      | http://www.w3.org/ns/dqv#                         |
| rdf      | http://www.w3.org/1999/02/22-rdf-syntax-ns        |
| rdfs     | http://www.w3.org/2000/01/rdf-schema#             |
| sd       | http://www.w3.org/ns/sparql-service-description   |
| void     | http://rdfs.org/ns/void                           |

[Dublin core terms](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/) are used to annotate dataset. The properties dct:created, dct:modified, dct:source are part of the KEES Language profile.


FromSPARQL service description, KEES application should 
recognize sd:Service, sd:endpoint, sd:feature.

Knowledgebase building activities MUST be traced using [PROV ontology](https://www.w3.org/TR/prov-overview/). KEES application should 
recognize dct:source, prov:wasGeneratedBy, prov:wasAttributedTo.

Trustability is enabled by attaching quality observation to the ingested graph. 
For this feature KEES language profile reuses the terms in [Data on the Web Best Practices: Data Quality Vocabulary](https://www.w3.org/TR/vocab-dqv/) with dqv:hasQualityAnnotation,  dqv:value, oa:body, oa:motivation

The following picture summarizes the main elements of the KEES language profile.

![uml](architecture/uml.png)


## RDF Store requirement
To Know the **provenance** of each statement, it is of paramount importance to get an idea about data quality. For this reason, KEES requires that all statements must have a fourth element that links to a data source. This means that, for practical concerns, the KEES knowledge base is a collection of quads, i.e. a triple plus a link to metadata.

Any RDF Store that provides with a SPARQL endpoint and QUAD support is potentially compliant with KEES. 


## KEES protocol and axioms
KEES application **SHOULD** follow the rules

### Knowledge Graph status

During knowledge base building and updates, KEES agents can declare the completion of a KEES cycle window


For instance to  declare that a RDF Store is ready to be safely queried, execute following SPARQL UPDATE statement:

```sparql
INSERT { ?service kees:published ?now }
WHERE { ?service a kees:KnowledgeGraph BIND( NOW() AS ?now) }
```

To check if a RDF Store is *safe*: 

```sparql
ASK { ?service kees:published [] }`
```

### Get knowledge graph last update

```sparql
SELECT (MAX(?updated) as ?lastUpdated) WHERE {
    ?g sd:name ?graphName ;
    dct:modified ?updated
} GROUP BY ?graphName
```


### Get the date of the last ingestion

```sparql
SELECT (MAX(?updated) as ?lastPublished) WHERE {
    ?g sd:name ?graphName; 
        prov:wasGeneratedBy/rdf:type kees:Ingestion ;
        dct:modified ?updated
} GROUP BY ?graphName
```


### Request s knowledge graph reboot

Sometime you need to signal that data in knowledge base needs (or will need) to be update. In this case you **SHOULD** use:

```sparql
INSERT { ?service prov:invalidetedBy [ a prov: Agent ] }
WHERE { ?service a kees:KnowledgeGraph }
```

To test if you should reboot the knowledge base:
```sparql
ASK { ?service a kees:KnowledgeGraph; prov:invalidetedBy [] }
```

The kees agent implementations can add restrictions on agent allowed to as KB invalidation.


### Knowledge base locking
At times, multiple processes may need access to the same knowledge graph, leading to potential locking scenarios. Ideally, agents should utilize the OS standard locking mechanism (e.g., flock). However, if this isn't feasible, a stopgap solution can be employed (though it's not entirely secure):

To establish an exclusive lock on the knowledge graph, consider using this pseudo-code:

```bash

function unlock {
    DELETE  { ?service kees:isLockedBy ?anything }
    WHERE  { ?service  a kees:KnowledgeGraph;  kees:isLockedBy ?anything }
}

function lock {
    local UNIQUE_URI=${1:-"urn:uuid:$(date +%s%N)$(echo $RANDOM)"}
    while : ; do
        INSERT { ?service kees:isLockedBy <$UNIQUE_URI> } 
        WHERE { FILTER NOT EXISTS { ?service a kees:KnowledgeGraph; kees:isLockedBy [] } } 
        if  ASK { ?service  a kees:KnowledgeGraph;  kees:isLockedBy <$UNIQUE_URI>}; then
            break
        else
            sleep 10
        fi 
    done
}


LOCKID=$(lock)
trap unlock EXIT  # ensure lock is removed also on script exit
    ... do your jobs
unlock
trap '' EXIT
```

Be sure to delete lock even in case of error or script exit

## Example of a dump of a KEES compliant Knowledge Graph

```turtle
@base <http://www.example/sparql/> .
@prefix dct: <http://purl.org/dc/terms/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix kees: <http://linkeddata.center/kees/v1#> .
@prefix sd: <http://www.w3.org/ns/sparql-service-description#> .
@prefix : <#>


# default graph
{
    [] a kees:KnowledgeGraph ;
        sd:endpoint <> ;
        sd:feature kees:Status, kees:Locking ;
        kees:isLockedBy <urn:kees:kb:isLockedBy> [ a kees:Activity ] ;
        kees:bootCompleted "2023-12-10T01:00:01Z"^^xsd:dateTime ;
        kees:learningCompleted "2023-12-10T01:01:01Z"^^xsd:dateTime ;
        kees:reasoningCompleted "2023-12-10T01:02:01Z"^^xsd:dateTime ;
        kees:enrichingCompleted "2023-12-10T01:03:01Z"^^xsd:dateTime ;
        kees:published "2023-12-10T01:04:01Z"^^xsd:dateTime .
}


<http:/example.org/.well-known/kees> {
    [] a sd:NamedGraph;
        sd:name <http:/example.org/.well-known/kees>
        dct:modified "2023-12-10T01:01:02Z"^^xsd:dateTime ;
        dct:source  <http:/example.org/kees.ttl> ;
        prov:wasGeneratedBy [ a kees:Ingestion ] ;
        dqv:hasQualityMeasurement [ dqv:value 0.99 ; dqv:isMeasurementOf kees:trustLevel] .
}


<http:/example.org/.well-known/void> {
    [] a sd:NamedGraph;
        sd:name <http:/example.org/.well-known/void> ;
        dct:modified "2023-12-10T01:02:02Z"^^xsd:dateTime ;
        dct:source  <http:/example.org/void.ttl> ;
        prov:wasGeneratedBy [ a kees:Ingestion ] ;
        dqv:hasQualityMeasurement [ dqv:value 0.75 ; dqv:isMeasurementOf kees:trustLevel] .

    ## here the example triple exposed by http:/example.org/void.ttl
    <http:/example.org/.well-known/void> a kees:KnowledgeBaseDescription ;
        foaf:primaryTopic  <http:/example.org/dataset1#dataset> .

    <http:/example.org/dataset1#dataset> 
        void:dataDump 
            <http:/example.org/dataset/part1.ttl> ,
            <http:/example.org/dataset/part2.ttl> .
    
    ## ...
}


<http:/example.org/dataset1#dataset> {
    [] a sd:NamedGraph;
        sd:name <http:/example.org/dataset1#dataset>
        dct:modified 
            "2023-12-10T01:04:03Z"^^xsd:dateTime , 
            "2023-12-10T01:05:04Z"^^xsd:dateTime ;
        prov:wasGeneratedBy [ a kees:Ingestion ] ;
        dct:source: 
            <http:/example.org/dataset/part1.ttl> , 
            <http:/example.org/dataset/part2.ttl> .

    # ... here triples merged from all data dumps
}

```