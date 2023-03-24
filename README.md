# How to ... for SPARQL Queries

This is my personal collection of working SPARQL queries that can serve as patterns for solutions for typical problems encounterd when writing SPARQL queries.

## How to get labels for results for grouped results

Problem:

When a query uses GROUP BY the variables can not be used directly to get their labels.

Solution:

Place the grouping in a subquery:

```SPARQL
PREFIX 

```