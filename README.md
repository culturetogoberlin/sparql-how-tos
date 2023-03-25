# How to ... for SPARQL Queries

This is my personal collection of working SPARQL queries that can serve as patterns for solutions for typical problems encounterd when writing SPARQL queries.

## How to get labels for results of grouped results

Problem:

When a query uses GROUP BY the variables can not be used directly to get their labels.

Solution:

Place the grouping in a subquery:

```SPARQL
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
# Pattern: Labelling bei Aggregationen über Subqueries
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