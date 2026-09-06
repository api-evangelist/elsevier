---
name: elsevier-fulltext-entitlement
description: Ask whether you may read an article before you try to fetch it, then retrieve the full text or its objects — the two-step that stops an agent from generating a wall of 403s.
api: Elsevier ScienceDirect APIs
generated: '2026-09-06'
method: generated
source: openapi/elsevier-sciencedirect-swagger.json, wadl/elsevier-article-entitlement.wadl, wadl/elsevier-article-retrieval.wadl
operations:
  - ArticleEntitlementRetrieval
  - ArticleRetrieval
  - ObjectRetrieval
  - ArticleMetadata
---

# Full text, and whether you are allowed to have it

## Check first

```
GET https://api.elsevier.com/content/article/entitlement/doi/{doi}
X-ELS-APIKey: <key>
Accept: application/json
```

`ArticleEntitlementRetrieval` exists precisely so you do not have to discover entitlement by
being refused. It accepts `doi`, `eid`, `pii`, `pubmed_id` or `scopus_id`, and it takes a
comma-separated list — batch the check, then fetch only what came back entitled.

This operation is **access-controlled**: it is not enabled on a new API key. Request enablement
from Elsevier API Support before building on it.

## Then retrieve

```
GET https://api.elsevier.com/content/article/doi/{doi}?view=FULL
X-ELS-APIKey: <key>
Accept: text/xml
```

`ArticleRetrieval` views, cheapest to richest: `META`, `META_ABS`, `META_ABS_REF`, `REF`,
`ENTITLED`, `FULL`. Ask for the narrowest view that answers your question — `FULL` on a large
article is a lot of XML and the quota is counted per request either way.

Representations include `text/xml`, `application/json`, `application/pdf` and
`application/rdf+xml`. PDF comes back as a `303` or `307` redirect with the document URL in the
`Location` header, so your client must follow redirects.

Article Retrieval is 50,000 requests per seven days at 10/second, and **unlimited for
text-mining API keys** — if that is your use case, say so when you register, because it changes
the economics completely.

## Figures and assets

`ObjectRetrieval` (`GET /content/object/doi/{doi}/ref/{ref}`) pulls the binary assets attached
to an article. Four resolutions per reference: base, `/high`, `/standard`, `/thumbnail`. Some
object responses are `202 Accepted` — the request is queued, not finished, so poll rather than
treating the empty body as an error.

## Metadata without the text

If all you need is discovery, `ArticleMetadata` (`GET /content/metadata/article`) is a
field-restricted search over full-text articles and is not access-controlled the way entitlement
is. Its total result set caps at 6,000 items.

## What the errors mean here

- `403` — you asked for content the institution does not subscribe to, or you are calling from
  outside its network. Not retryable. Get an Institutional Token if you are remote.
- `404` — the identifier does not resolve. Check the scheme; a DOI is not a PII.
- `500` — Elsevier's release notes record a historical class of 500s on documents not accessible
  for API use. Treat a persistent 500 on one identifier as an entitlement problem in disguise
  and quote `X-ELS-TransId` to support.

## Read the policy before you scale this

Text and data mining over ScienceDirect is governed by explicit provisions at
https://dev.elsevier.com/tdm_service.html — academic research only, delete the dataset when the
project ends, no competing derivative works. Every response also carries a `tdm-reservation`
header pointing at an ODRL policy. This is not boilerplate; it is the licence.
