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

### Items with Wikipedia articles in certain languages, missing an article in a specific language

Problem:

We want to get a list of items (of a certain kind) that have a Wikipedia article in certain languages, but not in specific language.

Use case:

Creating a checklist of items that need a Wikipedia article in my language. The fact that they are covered in - say - English, French, Italian and Spanish Wikipedia can be take as an indication that they are notable enough to have an article in my language.
