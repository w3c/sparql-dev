## Draft: PREFER, a new way to select preferred answers

## PREFER,ACCEPTLANG, hasLang(), PREFERLANG

## SEP-PREFER

## Authors

Jerven Bolleman

## Abstract



RDF and SPARQL have excellent support for multiple languages and this is widely used. However, there is a lack of selecting a preferred language given a set of possibilities. 

## Motivation

While technically possible in SPARQL 1.1. We would like to have an easier to write query syntax that is potentially easier to optimize for query engines. Even if the final syntax can be rewritten to SPARQL 1.1 equivalent constructs.



Given the requirement to select labels/properties in languages in an order of user preference we currently need to use the lang() and langmatches() functions either in a series of optional blocks with tests for boundness, or a subselect with an ordering.


```sparql
SELECT 
  ?thing 
  ?label
WHERE {
  OPTIONAL {
    ?thing rdfs:label ?label .
    FILTER(langMatches(lang(?label), “fr”)
  }
  OPTIONAL {
    FILTER(! BOUND(?label))
    ?thing rdfs:label ?label .
    FILTER(langMatches(lang(?label), “en”)
  }
  FILTER(BOUND(?label) && BOUND(?thing))
}
```
Gives an example of the use with the optional solution.
```sparql
SELECT
  ?thing
  ?label
WHERE
{ { 
 SELECT  
   ?thing 
   ?label 
   ?order
 WHERE
 { VALUES ( ?lang ?order ) {
             ( "fr" 1 )
             ( "en" 2 )}
   ?thing  rdfs:label  ?label
   FILTER langMatches(lang(?label), ?lang)
 }

 GROUP BY ?thing ?label ?order
 ORDER BY ?order
 LIMIT   1
 }
 }
```


Neither variety are easy to write, and neither can take into account preferential settings regarding languages from a browser or http header.



The proposal is to introduce a new keyword `PREFER` and functionality in the syntax and abstract algebra.



The same query as above would look like this in the proposed syntax.


```
SELECT ?label 

WHERE

{

  [] rdfs:label PREFER(?label FROM langMatches(lang(?label), "FR"),

                         langMatches(lang(?label), "EN"), 

                         langMatches(lang(?label), "NL")) .

}
```

We will also introduce a second new keyword `ACCEPTLANG`, to refer to taking the order from the accept-lang http header provided by the client and a new shorthand for testing if a value has a matching language string `hasLang(?label, "EN")`.


```sparql
SELECT 
  ?label 
WHERE
{
  [] rdfs:label PREFER(?label FROM hasLang(?label, "FR"),
                                   hasLang(?label, "EN"), 
                                   hasLang(?label, "NL"),
                                   hasLang(?label, ACCEPTLANG)) .
}
```


With a special shorthand


```sparql
SELECT 
  ?label 
WHERE
{
  [] rdfs:label PREFERLANG(?label, "FR", "EN", "NL", ACCEPTLANG)) .
}
```


The `PREFER` keyword introduces a new complexity for query engines to implement. While it does not introduce new complexity in terms of query expressiveness, the practical impact on the query engines is not small. Specifically it introduces a new manner to introduce an ad-hoc sorted join and reduction. On the other hand, `PREFER` is a general solution that works on any effective boolean value providing function. For example one can write


```sparql
SELECT 
  ?label 
WHERE
{
  [] rdfs:label PREFER(?label FROM "cafe au lait"@FR == ?label, "cappuccino"@EN == ?label, "koffie verkeerd"@NL == ?labe, strlen(?label)<20) .
}
```

The word `FROM` is selected because it indicates that assignment is in the unusual direction. 

A not addressed concern is where would `PREFER` be allowed? Only in the object position or would we allow it in the subject or predicate of the BGPs.

## Thoughts about query efficiency

There is a potential for query engines to optimise this kind of query if they can already push down filters on values lower into the stack.

## Evidence of consensus

None

## Backwards compatibility

Existing implementations do not understand this logic.








