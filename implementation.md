Implementing KEES 
=======================================

Please before continue, have a look to the [KEES definitions and principles](README.md)

This working draft collects a proposal for a concrete KEES implementation.


## KEES Vocabulary

The **KEES vocabulary** defines few new terms in the  http://linkeddata.center/kees/v1#  namespace ( usual prefix *kees:*). 
It consists of few OWL classes, mainly derived from existing ontologies. 

![uml](v1/images/uml.png)

`kees:KnowledgeBaseDescription`
: is a document that contains the description of the knowledge base with the purpose of publishing and transferring knowledge base building information. It is a subclass of a [void:DatasetDescription](https://www.w3.org/TR/void/#void-file)

`kees:BootActivity`, `kees:IngestionActivity`, `kees:ReasoningActivity`, `kees:EnrichingActivity`, `kees:PublishingActivity`, `kees:VersioningActivity`
: are subclasses of [prov:Activity](https://www.w3.org/TR/prov-o/#Activity) related to the KEES cycle windows

Besides classes, KEES vocabulary defines some individuals:

`kees:trust`
: it is a predefined  instance of [dqv:Metric](https://www.w3.org/TR/vocab-dqv/#dqv:Metric) to be used in a [dqv:QualityMeasurement](https://www.w3.org/TR/vocab-dqv/#dqv:QualityMeasurement) to define a trust level in a [dqv:UserQualityFeedback](https://www.w3.org/TR/vocab-dqv/#dqv:UserQualityFeedback)  according with the [Data Quality Vocabulary](https://www.w3.org/TR/vocab-dqv/). 

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

Knowledgebase building activities MUST be traced using [PROV ontology](https://www.w3.org/TR/prov-overview/). KEES application should 
recognize prov:wasDerivedFrom, prov:wasGeneratedBy, prov:wasAttributedTo.

Trustability is enabled by attaching quality observation to the ingested graph. 
For this feature KEES language profile reuses the terms in [Data on the Web Best Practices: Data Quality Vocabulary](https://www.w3.org/TR/vocab-dqv/) with dqv:hasQualityAnnotation,  dqv:value, oa:body, oa:motivation

The following picture summarizes the main elements of the KEES language profile.

![uml](architecture/uml.png)




## RDF Store requirement
To Know the **provenance** of each statement, it is of paramount importance to get an idea about data quality. For this reason, KEES requires that all statements must have a fourth element that links to a data source. This means that, for practical concerns, the KEES knowledge base is a collection of quads, i.e. a triple plus a link to metadata.

Any RDF Store that provides with a SPARQL endpoint and QUAD support is compliant with KEES. 

## KEES protocol
KEES application **SHOULD** follow the rules


### Knowledge base validity

During knowledge base building and update the knowledge base could be in an inconsistent state.
If a statement with the subject <urn:kees:kb> and the predicate *dct:valid* exists in the default graph, 
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

Sometime you need to signal that data in knowledge base needs (or will need) to be update. In this case you **SHOULD** use:

```sparql
INSERT DATA { <urn:kees:kb> prov:invalidetedBy <agent_uri> }
```

To test if you should reboot the knowledge base:
```sparql
ASK { <urn:kees:kb> prov:invalidetedBy [] }
```

The kees agent implementations can add restrictions on agent allowed to as KB invalidation.


### Knowledge base locking
Sometimes multiple process insist on the same knowledge graph. In this case locking can happens. If possible agent holud use OS standard locking mechanism (e.g. flock), but if this is not possible you can use following stop gap solution (warning it is not completelly safe):
To create an exclusive lock on the knowlegde graph use this pseudo code:
```bash

function unlock {
    DELETE  { <urn:kees:kb> <urn:kees:kb:isLockedBy> ?anything }
    WHERE  { <urn:kees:kb> <urn:kees:kb:isLockedBy> ?anything }
}

function lock {
    local UNIQUE_URI="urn:uuid:$(date +%s%N)$(echo $RANDOM)"
    while : ; do
        INSERT { <urn:kees:kb> <urn:kees:kb:isLockedBy> <$UNIQUE_URI> } 
        WHERE { FILTER NOT EXISTS { <urn:kees:kb> <urn:kees:kb:isLockedBy> [] }} 
        if  ASK {<urn:kees:kb> <urn:kees:kb:isLockedBy> <$UNIQUE_URI>} then
            break
        else
            sleep 10
        fi 
    done
    echo UNIQUE_URI
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
    <urn:kees:kb> 
        dct:created "2023-12-10T01:10:01Z"^^xsd:dateTime; # indicates when the knowledge base could be considered stable
        dct:valid "2023-12-10T01:15:01Z"^^xsd:dateTime; # indicates when the knowledge base could be considered safe
        kees:isLockedBy <urn:kees:kb:isLockedBy> [] # indicates when the knowledge base was locked when the Knowledge Graph was dumped
    .	
}

<http:/example.org/.well-known/kees> {
    [] a sd:NamedGraph;
        sd:name <http:/example.org/.well-known/kees>
        dct:created "2023-12-10T01:01:02Z"^^xsd:dateTime;
        dct:modified "2023-12-10T01:01:02Z"^^xsd:dateTime ;
        prov:wasDerivedFrom  <http:/example.org/kees.ttl> ;
        dqv:hasQualityMeasurement [
            a dqv:UserQualityFeedback;
            dqv:value 0.99;
            dqv:isMeasurementOf kees:trust
        ]
}

<http:/example.org/.well-known/void> {
    [] a sd:NamedGraph;
        sd:name <http:/example.org/.well-known/void>
        dct:created "2023-12-10T01:01:02Z"^^xsd:dateTime;
        dct:modified "2023-12-10T01:02:02Z"^^xsd:dateTime ;
        prov:wasDerivedFrom  <http:/example.org/void.ttl> ;
        dqv:hasQualityMeasurement [
            a dqv:UserQualityFeedback;
            dqv:value 0.75;
            dqv:isMeasurementOf kees:graphTrust
        ]
    .

    ## here the example triple exposed by http:/example.org/void.ttl
    <http:/example.org/.well-known/void> a kees:KnowledgeBaseDescription;
        foaf:primaryTopic  <http:/example.org/dataset1#dataset> .

    <http:/example.org/dataset1#dataset> 
        void:dataDump <http:/example.org/dataset/part1.ttl> ,<http:/example.org/dataset/part2.ttl>
    .
    
    ## ...
}

<http:/example.org/dataset1#dataset> {
    [] a sd:NamedGraph;
        sd:name <http:/example.org/dataset1#dataset>
        dct:created "2023-12-10T01:03:02Z"^^xsd:dateTime;
        dct:modified 
            "2023-12-10T01:04:02Z"^^xsd:dateTime, 
            "2023-12-10T01:05:02Z"^^xsd:dateTime;
        prov:wasDerivedFrom: 
            <http:/example.org/dataset/part1.ttl>, 
            <http:/example.org/dataset/part2.ttl> .

    # ... here triples merged from all datadumps
}

```