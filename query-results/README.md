# Query Samples

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