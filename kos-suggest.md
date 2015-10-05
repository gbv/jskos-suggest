# Introduction

**KOS Suggest** is an [OpenSearch Suggestions] compatible API to query
suggestions of concepts in Knowledge Organization Systems.

## Motivation

Knowledge Organization Systems (KOS), such as classifications, taxonomies,
thesauri, gazeteer, and ontologies, provide controlled vocabularies of distinct
concepts, each identified by an unique identifier. To select a concept without
knowing its identifier it requires methods to search or browse in the KOS.

Search suggestions, also known as autocomplete or type-ahead, provide a list of
recommended search terms as the user types a query. When searching in a KOS,
however, the user is not looking for a terms but for concepts that often
include multilingual and ambiguous labels.  KOS Suggest specifies methods to
query **concept suggestions** to be used as search suggestion or concept
selection in user interfaces.

The API is fully compatible with [OpenSearch Suggestions], so every [KOS
Suggest Service] can also be used as OpenSarch Suggestions service.

## Conformance requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119].

## Status of this document

KOS Suggest is currently being specified as part of project [coli-conc].  The
most current version is available at <http://gbv.github.io/jskos-suggest/>.

Feedback and open issues are collected in an issue tracker at
<https://github.com/gbv/jskos-suggest/issues>.

# Overview
[KOS Suggest Service]: #overview

A **KOS Suggest Service** consists of an HTTPS or HTTP URL that, when requested
via HTTP GET as defined by the [request format], returns concept suggestions as
defined by the [response format].

A response is basically created in two steps:

1. The [request parameters] **query** (or **query^**) and **type**
   (optional), possibly extended by request parameter **language** and [request
   header] **Accept-Language** are used to retrieve a list of matching concepts.

2. The list of concepts is mapped to a JSON array with a label, a description, 
   and an URI for each concept. Mapping is controlled by [language preference]
   and by [format strings] given in the optional request parameters **label** 
   and **description**.

# Request format
[request format]: #request-format

## Request headers
[request header]: #request-headers

The following HTTP request headers SHOULD be sent:

Accept
  : The value `application/json` or `application/x-suggestions+json`
User-Agent
  : An appropriate client name and version number
Accept-Language
  : A selection of preferred languages as defined by HTTP

## Request parameters
[request parameter]: #request-parameters
[request parameters]: #request-parameters

A KOS Suggest Service MUST respect the following URL query parameters:

query
  : string query
query^
  : prefix query
type
  : An optional concept type, given as URI string
language
  : preferred languages
label
  : An optional [format string] to build label strings from
description
  : An optional [format string] to build description strings from
callback
  : An optional callback function to enable [JSONP]
 
A request MUST include at most one of the parameters **query** and **query^**.
A KOS Suggest Service MUST respond with an [error response] if both are given.

<div class="example">
Search concepts that begin with "Mark":

    ?query^=Mark

Search concepts of type <http://example.org/person> with string "Müller":

    ?query=Müller&type=http://example.org/person

</div>

# Response format
[response format]: #response-format

The response format of KOS Suggest API equals to the response format of
[OpenSearch Suggestions] with some additional constraints.

A successful response is a HTTP response with status code 200 or 203. 

## Response headers

A successful response MUST include the following HTTP response header:

Content-Type
  : The value `application/json`, `text/javascript; charset=utf-8`, or
    `application/x-suggestions+json` for normal response. The value 
    `application/javascript` or `application/javascript; charset=utf-8`
    for [JSONP] response.

The following HTTP response header SHOULD also be included: to support
Cross-Origin Resource Sharing (CORS):

Access-Control-Allow-Origin
  : The value `*`

## Response body

The response body of a successful response MUST be a JSON array of four values:

query string
  : which MUST be a string created by normalizing one of the [request parameters]
    **query** or **query^**.
labels
  : which MUST be a JSON array of strings. The strings SHOULD NOT be empty.
descriptions
  : which MUST be a JSON array of strings.
identifiers
  : which MUST be a JSON array of distinct URIs given as strings

The **labels**, **descriptions**, and **identifiers** MUST have same number of members.

The response MUST be normalized to Unicode Normalization Form C (NFC).

The response array MUST be wrapped in a [JSONP] callback function if [request
parameter] **callback** was sent with a valid value.

<div class="example">
A minimal valid response body, typically returned in response to an empty query
parameter, consists of an empty query string and three empty arrays:

```json
["",[],[],[]]
```

The specification does not enforce a special kind of query string
normalization.  Reasonable methods can affect whitespace, diacritics, and case.
For instance the request

    ?query=Müller

could result in a response with query string normalized to `muller`:

```json
["muller",
  ["Herta Müller","Herr und Frau Müller"],
  ["German-Romanian novelist, poet and essayist", "Swiss activists"],
  ["http://www.wikidata.org/entity/Q38049", "http://www.wikidata.org/entity/Q1613934"]
]
```

To find the query string in labels for highlighting, a client needs to apply 
the same normalization (`herta muller`, `herr und frau muller`).
</div>

## Format strings
[format string]: #format-strings
[format strings]: #format-strings

**Format strings** are used by a KOS Suggest Service to map a concept to a
label string or to a description string. The mapping is further affected by
[language preference]. Format strings can be provided by [request parameter]
**label** or **description**.  A KOS Suggestions Service SHOULD use a default
format strings if one of these parameters are missing.

A format string consists of a string with embedded templates given in curly
brackets.  The string MUST match the following syntax, given in ISO EBNF:

```
format-string = { any-character | template } 
template      = "{", [ count ], fields, [ ":" delimiter ],  "}"
count         = "*" | ( digit - "0" ), { digit }
fields        = field, { "|", field }
field         = name [ "@" [ languages ] ]
name          = namechar, { namechar | digit }
namechar      = alpha | "_" | "."
languages     = language-tag, { "|", language-tag }
language-tag  = primary { "-", subtag }
primary       = alpha, [ alpha ], [ alpha ], [ alpha ], [ alpha ], [ alpha ], [ alpha ], [ alpha ] 
subtag        = anum, [ anum ], [ anum ], [ anum ], [ anum ], [ anum ], [ anum ], [ anum ] 
anum          = alpha | digit
delimiter     = { any-character - "}" }
alpha         = "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J" | "K" | "L" | "M" | 
                "N" | "O" | "P" | "Q" | "R" | "S" | "T" | "U" | "V" | "W" | "X" | "Y" | "Z" |
                "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j" | "k" | "l" | "m" |
                "n" | "o" | "p" | "q" | "r" | "s" | "t" | "u" | "v" | "w" | "x" | "y" | "z" 
digit         = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
```

For each template the following fields are extracted:

count
  : a positive integer or the character `*`. Set to `1` by default
    (template `{foo}` is equivalent to `{1foo}`).
fields
  : a list of field names
delimiter
  : an optional string, set to "`, `" by default
    (template `{5foo:, }` is equivalent to `{5foo}`).
    The empty string is allowed (for instance template `{*foo:}`).
languages
  : an optional list of language tags. The list can be empty
    (for instance template `{foo@}`) but the empty list differs
    from no list (template `{foo}`).

The template is then replaced by a string constructed from a concept as following:

<div class="note">
*...preliminary, informal description...*

* Languages are ignored if fields have no languages (e.g. `notation` or `uri`)
* language `@` (all languages) is equal to no language only if `count` is `1`:
  `{foo@}` is equal to `{foo}` but `{*foo@}` is not equal to `{*foo}`.
* For fields with languages:
    * If the `template` contains no `languages` field
        * Use the best matching [prefered language], if available
        * Use any other available language, otherwise
    * Use the specified language(s), if available
        * Select best matching to [language preference] if concept
          has multiple languages matching to the template
</div>

<div class="note">
Delimiters are be ignored if field **count** is `1`, so `{1foo:/}` is valid but
equivalent to `{foo}`.
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

## Language preference
[language preference]: #language-preference
[prefered language]: #language-preference

Knowledge Organization Systems are often multilingual. A KOS Suggests Service
MUST use language matching, as described in [RFC 4647] to select from sets of
languages.

The Language Priority List is constructed from request parameter **language**
(if given) or request header **Accept-Language** (otherwise).  If both are
given, the latter list MAY be appended to the former with less priority.

<div class="note">
This section requires additional description to distinguish language 
lookup (RFC section 3.4) and language filtering (RFC section 3.3.2)
</div>

Language preferences SHOULD also be used for querying, for instance to select
which languages to search in.

## JSONP

If [request parameter] **callback** was given, a successful response MUST be wrapped 
in the JavaScript callback function provided by this parameter.

A KOS Suggest Service MUST allow at least alphanumerical characters,
underscores, and dollar characters in callback function names. A KOS Suggest
Service SHOULD responds with an [error response] if the callback function name
contains other characters.

## Error response
[error response]: #error-response

A response with HTTP status code in the 4xx or 5xx range SHOULD include a
response body in JSON format with information about the error.

In particular, an error response with status code 422 MUST be sent in response
to invalid [request parameters]:

* Both **query** and **query^** provided
* Value of **type** is not an URI
* Value of **language** does not look like a `|` separated list of
  language codes
* Value of **label** or **description** does not look like a valid
  [format string]
* Value of **callback** contains disallowed characters

# References

## Normative References

* Berners-Lee, T., Fielding, R., Masinter, L. 2005: 
  "RFC 3986: Uniform Resource Identifiers (URI): Generic Syntax".
  <https://tools.ietf.org/html/rfc3986>.

* Phillips, A., Davis, M. 2006. "RFC 4647: Matching of Language Tags".
  <http://tools.ietf.org/html/rfc4647>.

* Bradner, S. 1997. "RFC 2119: Key words for use in RFCs to Indicate Requirement Levels".
  <http://tools.ietf.org/html/rfc2119>.

* Crockford, D. 2006. "RFC 6427: The application/json Media Type for JavaScript Object Notation (JSON)".
  <http://tools.ietf.org/html/rfc4627>.

* ISO/IEC 14977: 1996: "Extended Backus-Naur Form".

## Informative References

* Clintin, D. 2005, “OpenSearch Suggestions 1.0”
  <http://www.opensearch.org/Specifications/OpenSearch/Extensions/Suggestions>

* van Kesteren, Anne. 2014. "Cross-Origin Resource Sharing".
  <http://www.w3.org/TR/cors/>

* Voß, J. 2015. “JSKOS data format for Knowledge Organization Systems”.
  <https://gbv.github.io/jskos/>. 

* Voß, J. 2015. “JSKOS API Specification”.
  <https://gbv.github.io/jskos-api/>. 

[OpenSearch Suggestions]: http://www.opensearch.org/Specifications/OpenSearch/Extensions/Suggestions/1.0
[JSKOS]: https://gbv.github.io/jskos/
[RFC 2119]: https://tools.ietf.org/html/rfc2119
[RFC 4647]: https://tools.ietf.org/html/rfc4647
[coli-conc]: https://coli-conc.gbv.de/

