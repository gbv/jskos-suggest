# Introduction

**KOSuggest** is an [OpenSearch Suggestions]-compatible API to query
suggestions of concepts in Knowledge Organization Systems.

## Motivation

Knowledge organization systems (KOS), such as classifications, taxonomies,
thesauri, gazeteer, and ontologies, provide controlled vocabularies of distinct
concepts, each identified by an unique identifier. To select a concept without
knowing its identifier it requires methods to search or browse in the KOS.

Search suggestions, also known as autocomplete or type-ahead, provide a list of
recommended search terms as the user types a query. When searching in a KOS,
however, the user is not looking for a terms but for concepts that often
include multilingual and ambiguous labels.  KOSuggest specifies methods to
query **concept suggestions** to be used as search suggestion or concept
selection in user interfaces.

The API is fully compatible with [OpenSearch Suggestions], so every [KOSuggest
Service] can also be used as OpenSarch Suggestions service.

## Conformance requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119].

## Status of this document

KOSuggest is currently being specified as part of project [coli-conc].  The
most current version is available at <http://gbv.github.io/jskos-suggest/>.

Feedback and open issues are collected in an issue tracker at
<https://github.com/gbv/jskos-suggest/issues>.

# Overview
[KOSuggest Service]: #overview

A **KOSuggest Service** consists of an HTTPS or HTTP URL that, when requested
via HTTP GET as defined by the [request format], returns concept suggestions as
defined by the [response format].

The request is first turned into a sorted list of concepts which are then
mapped to pairs of label and description each.

# Request format
[request format]: #request-format

## Request headers

The following HTTP request headers SHOULD be sent:

Accept
  : The value `application/json` or `application/x-suggestions+json`
User-Agent
  : An appropriate client name and version number
Accept-Language
  : A selection of preferred languages as defined by HTTP

## Request parameters

A KOSuggest Service MUST respect the following URL query parameters:

query
  : string query
query^
  : prefix query
language
  : preferred language
type
  : An optional concept type, given as URI string
label
  : An optional [format string] to build label strings from
description
  : An optional [format string] to build description strings from
 
Only one of either string query (`query`) or prefix query (`query^`) MUST be sent.
A KOSuggest Service MUST respond with an [error response] if both are sent.

<div class="example">
Search concepts that begin with "Mark":

    ?query^=Mark

Search concepts of type <http://example.org/person> with string "Müller":

    ?query=Müller&type=http://example.org/person

</div>

# Response format
[response format]: #response-format

The response format of KOSuggest API equals to the response format of
[OpenSearch Suggestions] with some additional constraints.

A successful response is a HTTP response with status code 200 or 203. 

## Language resolution
[language preference]: #language-resolution
[prefered language]: #language-resolution

Knowledge organization systems are often multilingual but not all concepts have
unique labels in all languages. KOSuggest defines the following language
resolution algorithm:

If request parameter `language` is given, it MUST be used for language
resolution.  Otherwise the request header `Accept-Language` MUST be used for
language resolution.  If both are given, the request header `Accept-Language`
MAY be used as fallback only.

The language preferences SHOULD be used for querying, for instance to select
which languages to search in.

The language preferences MUST be used on [format string] application.

## Format strings
[format string]: #format-strings

Format strings are used to by a KOSuggest Service to map a concept and a
[language preference] to a label string or description string, respectively.

<div class="note">
The syntax of format strings has to be decided. Here is a draft in ISO EBNF:

```
format-string = { any-character | template } 
template      = "{", [ count ], fields, [ ":" delimiter ],  "}"
count         = "*" | ( digit - "0" ), { digit }
fields        = field, { "|", field }
field         = name [ "@" [ language ] ]
name          = alpha { alpha }
language      = ? language range as defined by RFC 4647 ?
delimiter     = { any-character - "}" }
alpha         = "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J" | "K" | "L" | "M" | 
                "N" | "O" | "P" | "Q" | "R" | "S" | "T" | "U" | "V" | "W" | "X" | "Y" | "Z" |
                "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j" | "k" | "l" | "m" |
                "n" | "o" | "p" | "q" | "r" | "s" | "t" | "u" | "v" | "w" | "x" | "y" | "z" 
digit         = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
```

* The default `count` is `1`, so `{foo}` is equivalent to `{1foo}`
* The default delimiter is `, `, so `{*foo:, }` is equivalent to `{*foo}`.
* The empty string is a valid delimiter can be the empty string (e.g. `{*foo:}`)
* Delimiters can be ignored if count is `1`, so `{1foo:/}` is valid but equivalent to `{foo}`
* Languages are ignored if fields have no languages (e.g. `notation` or `uri`)
* language `@` (all languages) is equal to no language only if `count` is `1`:
  `{foo@}` is equal to `{foo}` but `{*foo@}` is not equal to `{*foo}`.
* For fields with languages:
    * If the `template` contains no `language`
        * Use the best matching [prefered language], if available
        * Use any other available language, otherwise
    * Use the specified language(s), if available
        * Select best matching to [language preference] if concept
          has multiple languages matching to the template
</div>

<div class="example">
First notation, followed by preferred label:

    {notation}: {prefLabel}

The name followed by date of birth and date of death:

    {name} ({birthDate}-{deathDate})

All notations, separated by the string `", "` (default):

    {*notation}

The first three notations, separated by the string `"/"`:

    {3notation:/}

The first two definitions or examples (note that this is not equal to the first
two definitions or the first two examples!):

    {2definition|example}

All alternative labels in all languages, separated by the string `", "`:

    {*altLabel@}

List of alternative labels in Arabic or its ISO 693 language subtags, separated
by arabic comma `،` (U+060C):

    {*altLabel@ar:،}

</div>

## Response headers

...

## Response body

The response body of a successful response MUST be a JSON array of four values:

* **query string** which MUST be a string
* **labels** which MUST be a JSON array of strings
* **descriptions** which MUST be a JSON array of strings
* **identifiers** which MUST be a JSON array of distinct URIs given as strings

The **query string** MUST be a string. 

The **labels**, **descriptions**, and **identifiers** MUST have same number of members.

Members of the **labels** array SHOULD NOT be empty strings.

The response MUST further be normalized to Unicode Normalization Form C (NFC).

<div class="note">
A minimal valid response body, typically returned in response to an empty query
parameter, consists of an empty query string and three empty arrays:

```json
["",[],[],[]]
```
</div>

## Error response
[error response]: #error-response

A response with HTTP status code in the 4xx or 5xx range SHOULD include a
response body with information about the error.

# References

## Normative References

* Berners-Lee, T., Fielding, R., Masinter, L. 2005: 
  “RFC 3986: Uniform Resource Identifiers (URI): Generic Syntax”.
  <https://tools.ietf.org/html/rfc3986>.

* Phillips, A., Davis, M. 2006. “RFC 4647: Matching of Language Tags”.
  <http://tools.ietf.org/html/rfc4647>.

* Bradner, S. 1997. “RFC 2119: Key words for use in RFCs to Indicate Requirement Levels”.
  <http://tools.ietf.org/html/rfc2119>.

* Crockford, D. 2006. “RFC 6427: The application/json Media Type for JavaScript Object Notation (JSON)”.
  <http://tools.ietf.org/html/rfc4627>.

* van Kesteren, Anne. 2014. “Cross-Origin Resource Sharing”.
  <http://www.w3.org/TR/cors/>

## Informative References

* Clintin, D. 2005, “OpenSearch Suggestions 1.0”
  <http://www.opensearch.org/Specifications/OpenSearch/Extensions/Suggestions>

* Voß, J. 2015. “JSKOS data format for knowledge organization systems”.
  <https://gbv.github.io/jskos/>. 

* Voß, J. 2015. “JSKOS API Specification”.
  <https://gbv.github.io/jskos-api/>. 

[OpenSearch Suggestions]: http://www.opensearch.org/Specifications/OpenSearch/Extensions/Suggestions/1.0
[JSKOS]: https://gbv.github.io/jskos/
[RFC 2119]: https://tools.ietf.org/html/rfc2119
[coli-conc]: https://coli-conc.gbv.de/

