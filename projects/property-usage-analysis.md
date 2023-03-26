# Notizen zum Projekt: Property-Verwendung analysieren

## Zielsetzung

Erhebung von statistischen Daten zur Verwendung von Properties bei bestimmten Items

- Welche Properties werden überhaupt mit den betreffenden Items verknüpft?
- Wie oft sind die Properties bei diesen Items verwendet?
- Gibt es bei Unterklassen der untersuchten Items Unterschiede in der Verwendung von Properties?

## Arbeitsnotizen

nächste Tasks: 

- VALUES für die Festlegung der Item-Typen (und Logik der Item-Typen)

Stand: 

```SPARQL
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
SELECT ?prop ?propLabel ?oCount 
  WHERE {
  {
    SELECT ?prop (COUNT (?o) AS ?oCount)
    WHERE {
      ?s ?pd ?o.
      ?prop wikibase:directClaim ?pd .
      ?s wdt:P31/wdt:P279* wd:Q3305213 .
    }
    GROUP BY ?prop
  }
  ?prop rdfs:label ?propLabel .
  FILTER((LANG(?propLabel)) = "en")
}
ORDER BY DESC(?oCount)

```
[Try it on QLever](https://qlever.cs.uni-freiburg.de/wikidata/jmhFw7)

Version mit VALUES
(aktuell Kapazitätsprobleme bei QLever)

```SPARQL
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
SELECT ?prop ?propLabel ?oCount 
  WHERE {
  {
    SELECT ?prop (COUNT (?o) AS ?oCount)
    WHERE {
	VALUES ?type { wd:Q3305213 }
      ?s ?pd ?o.
      ?prop wikibase:directClaim ?pd .
      ?s wdt:P31/wdt:P279* ?type.
    }
    GROUP BY ?prop
  }
  ?prop rdfs:label ?propLabel .
  FILTER((LANG(?propLabel)) = "en")
}
ORDER BY DESC(?oCount)

```

## Back Log

Wäre interessant, ein JS-Tool zu bauen, das die regelmäßige Abfrage erlaubt, um Veränderungen ind er Propertie-Verwendung zu erfassen.