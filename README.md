# How to ... for SPARQL Queries

This is my personal collection of working SPARQL queries that can serve as patterns for solutions for typical problems encounterd when writing SPARQL queries.

## How to get labels for items in grouped results

Problem:

When a query uses GROUP BY the variables can not be used directly to get their labels.

Solution:

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

## How to statistically analyze the properties used in a certain type of item

Problem:

The use of properties in the statements documenting items is very heterogeneous. To get an idea of which properties are used to characterize certain items, we need statistical data about the usage of properties. The following query lists all properties occurring in statements about paintings and their usage in descending order.

Solution:

```SPARQL
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
# Count how often properties are use with items of tye painting
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

NOTE: This is a very costly query that can an will often cause time-outs. Samples of results are availabele in [/query-results/property-usage](/query-results/property-usage).