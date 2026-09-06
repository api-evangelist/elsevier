---
name: elsevier-citation-metrics
description: Get cited-by counts and year-by-year citation overviews for a set of documents, and understand what the Citation Overview API will and will not tell you.
api: Elsevier Scopus APIs
generated: '2026-09-06'
method: generated
source: openapi/elsevier-metadata-swagger.json, wadl/elsevier-abstract-citation-count.wadl, wadl/elsevier-abstract-citation.wadl, https://dev.elsevier.com/support.html
operations:
  - AbstractCitationCount
  - CitationsOverview
---

# Citation counts and overviews

## The cheap one: cited-by count

```
GET https://api.elsevier.com/content/abstract/citation-count?doi=10.1016/S0014-5793(01)03313-0
X-ELS-APIKey: <key>
Accept: application/json
```

`AbstractCitationCount` accepts `doi`, `scopus_id`, `pii` or `pubmed_id`, and takes a
comma-separated list, so batch your identifiers rather than looping. This is the most generous
quota on the platform — 50,000 requests per seven days at 10/second — which is why it is the
right operation for bulk work.

It also has an HTML rendering: add `httpAccept=text/html` and the response is a badge you can
drop straight into a page. That is what Elsevier's own cited-by embed does.

## The gated one: citation overview

`CitationsOverview` (`GET /content/abstract/citations`) returns citations broken down by year,
with or without self-citations, across a date range.

Two things to know before you plan around it:

1. **It is access-controlled.** It is not enabled on a new key. You have to write to Elsevier
   API Support with your key, institution, use case and requested quota, and Elsevier states it
   "cannot guarantee permission".
2. **It does not return the citing articles.** Elsevier's own FAQ calls this out as a common
   misconception. You get counts, not a list. If you need the works that produced those counts,
   that is Scopus Search with a `REFEID` query — and Elsevier restricts that to subscribers for
   specific use cases too.

## Budget

- `AbstractCitationCount`: 50,000 / 7 days, 10 req/s
- `CitationsOverview`: 20,000 / 7 days, 4 req/s, once enabled

## Errors worth branching on

`403` here usually means the operation is not enabled on your key rather than that the document
is out of reach — check `X-ELS-Status` before assuming an entitlement problem with the content.
