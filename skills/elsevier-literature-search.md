---
name: elsevier-literature-search
description: Run a bounded literature search across Scopus (indexed metadata) or ScienceDirect (full text) and page through the results without burning the weekly quota.
api: Elsevier Research Products APIs
generated: '2026-09-06'
method: generated
source: openapi/elsevier-search-swagger.json, openapi/elsevier-sciencedirect-swagger.json, https://dev.elsevier.com/api_key_settings.html
operations:
  - ScopusSearch
  - ScienceDirectSearchV2
  - AffiliationSearch
  - AuthorSearch
---

# Searching Elsevier literature

## Pick the right index first

Scopus and ScienceDirect are different databases and the choice is not cosmetic.

- **Scopus** (`GET /content/search/scopus`) indexes abstracts, references and citations across
  thousands of publishers. Use it for citation analysis, author and affiliation work, and
  coverage beyond Elsevier.
- **ScienceDirect** (`GET /content/search/sciencedirect`) searches full text, mostly Elsevier's
  own journals and books. Use it when you need the words inside the article.

Scopus-native identifiers (AF-ID, Scopus ID) do not work against ScienceDirect.

## Call it

```
GET https://api.elsevier.com/content/search/scopus?query=TITLE-ABS-KEY(crispr)&count=25&start=0
X-ELS-APIKey: <key>
Accept: application/json
```

`query` is required and takes the product's boolean syntax — the field prefixes are documented
at https://dev.elsevier.com/tips/ScopusSearchTips.htm (and `AffiliationSearchTips.htm`,
`AuthorSearchTips.htm` for the sibling operations). Sending a bare keyword string works but
matches far more broadly than you probably want.

## Paging, and the wall you will hit

`start` and `count` page the result set. `count` defaults to 25 and maxes at 200 for the
STANDARD view; the COMPLETE and COMPONENT views cap at 25 and require an entitled key.

**`start`/`count` paging stops at 5,000 items on Scopus Search and 6,000 on ScienceDirect
Search V2.** Past that you must switch to cursor paging: send `cursor=*` on the first call and
then the `cursor` value returned in each response. Cursor paging is forward-only — you cannot
jump to an arbitrary page, and you cannot go back.

Read `opensearch:totalResults` before you start iterating. If it is larger than the cap and you
are not using a cursor, narrow the query instead of paging blindly.

## Budget the calls

Quota is per API, not per key. Scopus Search is 20,000 requests per rolling seven days at 9
requests/second; ScienceDirect Search V2 is 20,000 per seven days but only **2** per second.
Author Search is 5,000 per seven days at 2/second, which is the tightest search budget on the
platform.

Check `X-RateLimit-Remaining` on every response. When the weekly quota is gone that header
**stops appearing at all** and you get HTTP 429 with `X-ELS-Status: QUOTA_EXCEEDED` plus
`X-RateLimit-Reset`, a Unix timestamp. There is no `Retry-After`, so throttle yourself.

## What can go wrong

- `401` with `X-ELS-Status: AUTHENTICATION_ERROR` — the key is missing or wrong.
- `403` — the key is fine and the entitlement is not. Almost always this means you are calling
  from outside the subscribing institution's IP range, or asking for a view your key was never
  enabled for. Fix it with an Institutional Token, not by retrying.
- `429` — see above. Distinguish quota from throttle by whether `X-RateLimit-*` is still present.
- `400` — query syntax. Not retryable unchanged.

Full catalogue: `errors/elsevier-problem-types.yml`.

## Do not

Do not put the key in the query string, even though Elsevier's own examples do. Use the
`X-ELS-APIKey` header.
