---
name: elsevier-author-profile
description: Resolve a researcher from an ORCID iD or a name to a Scopus author profile, then pull their publications and SciVal metrics — and handle the duplicate-profile problem that this data has by design.
api: Elsevier Scopus APIs, Elsevier SciVal API
generated: '2026-09-06'
method: generated
source: openapi/elsevier-scopus-swagger.json, openapi/elsevier-scival-swagger.json, https://dev.elsevier.com/support.html
operations:
  - getAuthorByORCID
  - AuthorSearch
  - AuthorRetrievalid
  - AuthorRetrieval
  - ScopusSearch
  - authorMetrics
---

# Resolving a researcher

## Start from an ORCID if you have one

```
GET https://api.elsevier.com/analytics/scival/author/orcid/{orcid}
X-ELS-APIKey: <key>
```

`getAuthorByORCID` is the only deterministic entry point into this graph. Everything else is a
name match against algorithmically generated profiles.

## Otherwise search, and expect duplicates

`AuthorSearch` (`GET /content/search/author`) takes the Scopus author query syntax
(`AUTHLASTNAME(...) AND AUTHFIRST(...) AND AFFIL(...)`).

Elsevier is explicit that author profiles are built by an automated process and that the
matching algorithm deliberately errs toward creating a *new* profile rather than merging into
the wrong one. **One researcher routinely has several author IDs, and that is expected
behaviour, not a bug.** Plan for a set, not a single id. Corrections happen through the Scopus
web UI, not the API — you cannot fix this over HTTP.

Also note the quota: Author Search is 5,000 requests per seven days at **2 requests/second**,
the tightest budget on the platform. Do not use it as a lookup inside a loop.

## Pull the profile

`AuthorRetrievalid` (`GET /content/author/author_id/{author_id}`) returns the profile. The
`view` parameter drives both the shape and the entitlement — `ENHANCED` and `METRICS` views need
a subscribing key. Default is `STANDARD`.

## Their publications

Do not try to read a document list out of the author profile. Query Scopus Search instead:

```
GET https://api.elsevier.com/content/search/scopus?query=AU-ID(7004212771)&count=200
```

Page it exactly as in the `elsevier-literature-search` skill, including the 5,000-item cap.

## Their metrics

`authorMetrics` (`GET /analytics/scival/author/metrics`) takes `authors` (a comma-separated list
of Scopus author ids), `metricTypes`, `yearRange` and `includeSelfCitations`. It is SciVal, which
means it needs a SciVal subscription — the free non-commercial tier excludes the SciVal APIs by
name. Quota is 5,000 per seven days at 6/second.

## Errors

`403` on the SciVal call after a working Scopus call is the normal signal that the key has
Scopus but not SciVal. They are separate entitlements on the same key.
