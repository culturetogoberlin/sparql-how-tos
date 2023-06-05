# How to ... for SPARQL Queries

This is my personal collection of working SPARQL queries that can serve as patterns for solutions for typical problems encounterd when writing SPARQL queries.

## Problems and Solutions

### How to get info about items with knwon IDs

Problem:

We have a list of items with known IDs (Q Numbers) and want to get some information about them.

Solution (using Wikidata Query Service):

```SPARQL
# Getting information about some Wikidata items with known IDs
SELECT ?item ?itemLabel ?property ?value ?valueLabel
WHERE {
  VALUES ?item {
    wd:Q4176      # Cologne Kathedral
    wd:Q250212    # Freiburg Kathedral
  }
  VALUES ?property {
    wdt:P17       # Country (situated in)
    wdt:P825      # Dedicated to
  }
  OPTIONAL { ?item ?property ?value . }
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
[Try on WDQS](https://w.wiki/6dgb)

ToDos:

Labelling of properties.

### How to query for items that miss a statement with a certain property

Problem:

In order to know if a certain property is used normally/sparsely/never ... for a certain type of items we want to query for items that miss a statement with a certain property.

```SPARQL
# Paintings in Danish, Swedish, or Swiss collections that have no statement about the materials used
SELECT ?item ?itemLabel
WHERE 
{
  VALUES ?country {
    wd:Q35
    wd:Q34
    wd:Q39
  }
  ?item wdt:P31 wd:Q3305213 .
  ?item wdt:P276/wdt:P17 ?country .
  FILTER ( NOT EXISTS { ?item  wdt:P186 [] } )

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
[Try it on WDQS](https://w.wiki/6njz)

Note: The limitation to paintings in Danish, Swedish, or Swiss collections is only done to avoid timeout.

If we query without the filter for the missing statement ([Try it on WDQS](https://w.wiki/6nkA)) we get all items and can estimate the likelihood an item is qualified by such a statement (in this case, as of June 2023: ca. 39%).

ToDos:

Use COUNT() to get the number of items with and without the statement.

### How to query for items that meet condition A but not condition B

Problem:

We want to narrow down the results of a query by excluding items that meet a certain condition.

Solution (using Wikidata Query Service):

The following query from the [sample query manual](https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/queries/examples#All_Wikipedia_sites) returns all 331 (as of 5/15/2023) language versions of Wikipedia.

```SPARQL
# Get all Wikipedia sites
SELECT ?item ?itemLabel ?website
WHERE 
{
  ?item wdt:P856 ?website.
  # ?item wdt:P31 wd:Q10876391 .
  # FILTER NOT EXISTS { ?item wdt:P31 wd:Q10876391 . }
  # MINUS { ?item wdt:P31 wd:Q10876391 . }
  ?website wikibase:wikiGroup "wikipedia".
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
[Try it on WDQS](https://w.wiki/6huK)

If we add the condition that the site has to be a Wikipedia language version (Q10876391), we get only 329 (as of 5/15/2023) results.

```SPARQL
# All Wikipedia sites
# Get all Wikipedia sites
SELECT ?item ?itemLabel ?website
WHERE 
{
  ?item wdt:P856 ?website.
  ?item wdt:P31 wd:Q10876391 .
  # FILTER NOT EXISTS { ?item wdt:P31 wd:Q10876391 . }
  # MINUS { ?item wdt:P31 wd:Q10876391 . }
  ?website wikibase:wikiGroup "wikipedia".
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
[Try it on WDQS](https://w.wiki/6huP)

How can this be? What is different about the two results that are not Wikipedia language versions? The following query returns results that do not meet this additional criterion:

```SPARQL
# All Wikipedia sites
# Get all Wikipedia sites
SELECT ?item ?itemLabel ?website
WHERE 
{
  ?item wdt:P856 ?website.
  #?item wdt:P31 wd:Q10876391 .
  FILTER NOT EXISTS { ?item wdt:P31 wd:Q10876391 . }
  # MINUS { ?item wdt:P31 wd:Q10876391 . }
  ?website wikibase:wikiGroup "wikipedia".
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
[Try it on WDQS](https://w.wiki/6huS)

<small>The Norwegian Wikipedias is a special case because it is an "umbrella term for the two separate Norwegian-language editions of Wikipedia (Nynorsk - nn - and Bokmål/Riksmål - nb) which shared a single wiki until the creation of - nn - and redefinition of - no - as Bokmål/Riksmål only – prefix 'nb' redirects to 'no'" - description of Q191769.</small>

Both, FILTER NOT EXISTS and MINUS do the job, here with identical results. On the differences see [SPARQL Recommendation, Query Language #Negation](https://www.w3.org/TR/sparql11-query/#negation).

ToDos:

Explain the difference between FILTER NOT EXISTS and MINUS.

### How to get labels for items in grouped results

Problem:

When a query uses GROUP BY the variables can not be used directly to get their labels.

Solution (using QLever):

Place the grouping in a subquery:

```SPARQL
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
# Pattern: Labelling of aggregated items
# Example: List all German collections that are indicated in works of art
# and count the works for each collection; result in descending order.
SELECT ?collectionLabel ?itemCount WHERE { 
  { 
    SELECT ?collection (COUNT (?item) AS ?itemCount) WHERE {
      ?item wdt:P31/wdt:P279* wd:Q838948 .
      ?item wdt:P195 ?collection .
      ?collection wdt:P17 wd:Q183 .
    }
    GROUP BY ?collection
  }
  ?collection rdfs:label ?collectionLabel FILTER (LANG(?collectionLabel ) = "en") .
}
ORDER BY DESC(?itemCount) 

```
[Try it on QLever](https://qlever.cs.uni-freiburg.de/wikidata/qXYlja)

### How to statistically analyze the properties used in a certain type of item

Problem:

The use of properties in the statements documenting items is very heterogeneous. To get an idea of which properties are used to characterize certain items, we need statistical data about the usage of properties. The following query lists all properties occurring in statements about paintings and their usage in descending order.

Solution (using QLever):

```SPARQL
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
# Count how often properties are used with items of tye painting
SELECT ?prop ?propLabel ?oCount 
  WHERE {
  {
    SELECT ?prop (COUNT (?o) AS ?oCount)
    WHERE {
	VALUES ?type { 
      wd:Q3305213 # painting
    }
	  ?s wdt:P31 ?type.
      ?s ?pd ?o.
      ?prop wikibase:directClaim ?pd .
    }
    GROUP BY ?prop
  }
  ?prop rdfs:label ?propLabel .
  FILTER(LANG(?propLabel) = "en")
}
ORDER BY DESC(?oCount)

```
[Try it on QLever](https://qlever.cs.uni-freiburg.de/wikidata/8h9vxg)

NOTE: This is a very costly query that can and will often cause timeouts. Samples of results are availabele in [/query-results/property-usage](/query-results/property-usage).

## Queue - Problems that wait for a solution

### classes that are direct or indirect subclasses of a certain class

Problem:

List all classes that are direct or indirect subclasses of a certain class.

### Items with Wikipedia articles in certain languages, missing an article in a specific language

Problem:

We want to get a list of items (of a certain kind) that have a Wikipedia article in certain languages, but not in specific language.

Use case:

Creating a checklist of items that need a Wikipedia article in my language. The fact that they are covered in - say - English, French, Italian and Spanish Wikipedia can be take as an indication that they are notable enough to have an article in my language.
