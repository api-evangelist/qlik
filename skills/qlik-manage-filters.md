---
name: qlik-manage-filters
description: Perform the typical CRUD workflow for report filters in a Qlik app.
api: openapi/apps.json
operations:
- filtersList
- filtersCreate
- filtersGet
- filtersUpdate
- filtersDelete
generated: '2026-10-03'
method: generated
generator: extract-docs-artifacts.py skills (local)
source: openapi/apps.json ; every operationId checked against the contract
---

# qlik-manage-filters

Perform the typical CRUD workflow for report filters in a Qlik app.

## Steps

1. 1. `filtersList` – retrieve the list of filters (supports the `limit` pagination query parameter).
2. 2. `filtersCreate` – create a new filter (requires the request body fields defined in the contract).
3. 3. `filtersGet` – fetch a specific filter by its `{id}` path parameter.
4. 4. `filtersUpdate` – modify an existing filter using its `{id}` path parameter and the request body fields.
5. 5. `filtersDelete` – remove a filter identified by its `{id}` path parameter.

## Rules

- Pagination: use the `limit` query parameter to control page size.
- Rate limiting: no explicit limit is documented; on exhaustion the API returns no specific HTTP status.
