---
name: qlik-evaluation-run-and-download
description: Queue an app evaluation, retrieve its result, and download the detailed XML log.
api: openapi/apps.json
operations:
- evaluation#queueEvaluation
- evaluation#getOneEvaluation
- evaluation#downloadOneEvaluation
generated: '2026-10-03'
method: generated
generator: extract-docs-artifacts.py skills (local)
source: openapi/apps.json ; every operationId checked against the contract
---

# qlik-evaluation-run-and-download

Queue an app evaluation, retrieve its result, and download the detailed XML log.

## Steps

1. `evaluation#queueEvaluation` – requires the app GUID in the path and the evaluation request body.
2. `evaluation#getOneEvaluation` – uses the evaluation ID returned from the queue call.
3. `evaluation#downloadOneEvaluation` – uses the same evaluation ID to download the XML log.

## Rules

- (none stated)
