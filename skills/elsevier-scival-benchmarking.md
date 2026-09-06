---
name: elsevier-scival-benchmarking
description: Benchmark institutions, countries or research topics with SciVal metrics — the entity-then-metrics pattern the whole SciVal API follows.
api: Elsevier SciVal API
generated: '2026-09-06'
method: generated
source: openapi/elsevier-scival-swagger.json, wadl/elsevier-scival-institution.wadl, https://dev.elsevier.com/api_key_settings.html
operations:
  - institutionSearch
  - getInstitution
  - institutionMetrics
  - countryMetrics
  - topicMetrics
  - getTopicsByInstitutionId
  - worldMetrics
---

# Benchmarking with SciVal

## Everything here is a two-step

SciVal never lets you go from a name to a number in one call. You resolve an entity id, then ask
for metrics on that id. The pattern is identical for every entity type — institution, country,
country group, institution group, publication, source, subject area, topic, topic cluster.

```
GET https://api.elsevier.com/analytics/scival/institution/search?query=name(Delft)
GET https://api.elsevier.com/analytics/scival/institution/metrics?institutionIds=508076&metricTypes=ScholarlyOutput,FieldWeightedCitationImpact&yearRange=5yrs
X-ELS-APIKey: <key>
```

Base path is `/analytics/scival`, not `/content`. SciVal paginates with `offset`/`limit`, **not**
`start`/`count` — the content APIs and the analytics APIs do not share their pagination
vocabulary, and mixing them up is the most common integration mistake here.

## Metrics come in a bundle

`metricTypes` is a comma-separated list. Common ones: `ScholarlyOutput`, `CitationCount`,
`CitationsPerPublication`, `FieldWeightedCitationImpact`, `Collaboration`,
`CollaborationImpact`, `AcademicCorporateCollaboration`, `OutputsInTopCitationPercentiles`.
`byYear=true` returns the series instead of the aggregate; `includeSelfCitations=false` strips
self-citations.

Max 100 results per Metrics call. Total results are unbounded but vary by resource.

## Topics

`getTopicsByInstitutionId` returns an institution's key contributing topics — added in the
2019-09-12 release. Topics roll up into topic clusters, and both accept the same metrics
parameters, so you can benchmark an institution's strength inside a research area rather than
in aggregate.

`worldMetrics` gives you the global baseline to compare any of it against.

## Entitlement

SciVal is the one product Elsevier carves out of the free non-commercial tier by name. A key
that happily searches Scopus will return `403` on every path under `/analytics/scival` unless
the institution subscribes to SciVal. Test that first rather than debugging your query.

## Budget

Every SciVal lookup shares the same allowance: 5,000 requests per seven days at 6/second.
Because the pattern is two calls per answer, plan on roughly 2,500 benchmarks a week.
