---
name: Open and work a direct supplier incident in Agile Process Teams
description: >-
  Use the TraceLink OPUS event API to open a direct supplier incident, comment on it,
  submit it to the trade partner, read its activity history, and close it.
api: OPUS Platform Event API
surface: single-endpoint JSON RPC over HTTP POST
endpoint: https://opus.tracelink.com/api/events
operations:
  - agile-process-teams:add-direct-supplier-incident:v3
  - agile-process-teams:edit-direct-supplier-incident:v3
  - agile-process-teams:add-comment-for-incident:v1
  - agile-process-teams:list-comments-for-incident:v1
  - agile-process-teams:submit-direct-supplier-incident-to-partner:v1
  - agile-process-teams:get-activity-history-for-incident:v1
  - agile-process-teams:close-direct-supplier-incident:v2
  - agile-process-teams:reopen-direct-supplier-incident:v1
generated: '2026-08-02'
method: generated
x-authored-by: API Evangelist enrichment pipeline
source: https://github.com/tracelink/code-samples/blob/main/EventNames.MD
---

# Open a direct supplier incident

The OPUS Platform exposes **one** endpoint, `/api/events`, and **one** HTTP method,
`POST`. The operation is not the URL — it is the `eventName` inside the request body.
Every event name below is copied verbatim from TraceLink's own
[`EventNames.MD`](https://github.com/tracelink/code-samples/blob/main/EventNames.MD);
do not construct an event name you have not seen there.

## Envelope

Every request and response is `{ "header": {...}, "payload": {...} }`.

Required header fields (see `conventions/tracelink-conventions.yml`):

| field | value |
|---|---|
| `headerVersion` | `1` |
| `eventName` | the fully qualified `<app>:<action>:v<N>` name |
| `ownerId` | the Owner company id |
| `processNetworkId` | the network containing the process |
| `appName` | `"agile-process-teams"` |
| `dataspace` | `"default"` |

## Authentication

Two steps, once (see `authentication/tracelink-authentication.yml`):

1. With the short-lived browser session token as a **Bearer** token, POST
   `authorization-manager:generate-apiKeyCredentials:v1` to exchange it for a long-lived
   `apiKey` / `apiSecret` pair.
2. From then on, send `Authorization: Basic base64(apiKey + ":" + apiSecret)`. The pair
   does not expire — store it as a secret.

## Steps

1. **Open the incident** — `agile-process-teams:add-direct-supplier-incident:v3`.
   Payload templates for the minimum-field and all-field variants are in
   [`payload_samples/`](https://github.com/tracelink/code-samples/tree/main/payload_samples).
   Key payload members: `aptBusinessObjectSummary`, `aptBusinessObjectDescription`,
   `deviationType`, `materialType`, `materialSubtype`, `materialProblem`, and the
   `directSupplierImpact` object carrying `businessImpact` and `businessPriority`.
2. **Read the response header.** `header.isErr` tells you whether it worked;
   `header.errCode` carries the result code (`200_OK` on success);
   `header.licensePlate` is the correlation id to quote to support. The created object's
   identifier comes back on the payload — carry it into every subsequent call.
3. **Add context** — `agile-process-teams:add-comment-for-incident:v1`, and read them
   back with `agile-process-teams:list-comments-for-incident:v1`.
4. **Correct before escalating** — `agile-process-teams:edit-direct-supplier-incident:v3`
   for base-state or partner-impact edits.
5. **Send it to the partner** —
   `agile-process-teams:submit-direct-supplier-incident-to-partner:v1`. This is the step
   that makes the incident visible outside your company; treat it as irreversible.
6. **Audit** — `agile-process-teams:get-activity-history-for-incident:v1` returns the
   append-only history for the object.
7. **Close** — `agile-process-teams:close-direct-supplier-incident:v2`, and
   `...:reopen-direct-supplier-incident:v1` if it comes back.

## Version pinning

Operations version independently. In this same app `add` and `edit` are at `v3`, `close`
and `copy` at `v2`, and `reopen`, `submit`, and every shared comment/history operation at
`v1`. Pin the version per event, never per app.

## Retries

There is no idempotency key. Re-POSTing `add-direct-supplier-incident` after a timeout
can create a second incident. On an ambiguous failure, read the incident list or activity
history first and only then retry.

## Same operations over GraphQL

The identical actions are reachable at `https://opus.tracelink.com/api/graphql` through
the generic dispatch fields `genericActionCall(action, payload)` and `genericGetObject`,
with `Dataspace`, `companyId`, and `processNetworkId` sent as HTTP headers instead of
body-header fields. Working samples:
[`GraphQL/`](https://github.com/tracelink/code-samples/tree/main/GraphQL). Introspection
is authentication-gated, so the SDL is not published.
