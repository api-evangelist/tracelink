---
name: Verify a TraceLink serial number and read its hierarchy
description: >-
  Verify a serialized pharmaceutical product against TraceLink Serialized Operations
  Manager, then read the entity and its aggregation hierarchy to confirm what was
  actually shipped.
api: wsdl/tracelink-serialized-operations-manager.wsdl
surface: SOAP 1.1 over HTTPS
endpoint: https://api.tracelink.com/soap/som
operations:
  - SerialNumberVerification
  - GetSerializedEntity
  - GetSerializedEntityWithProduct
  - GetSerialNumberHierarchy
  - FetchSerialNumberHierarchyResult
  - GetLotStatus
generated: '2026-08-02'
method: generated
x-authored-by: API Evangelist enrichment pipeline
source: wsdl/tracelink-serialized-operations-manager.wsdl
---

# Verify a serial number in TraceLink

TraceLink Serialized Operations Manager (SOM) is a SOAP service. There is no OpenAPI —
the contract is the WSDL at `https://api.tracelink.com/soap/som?wsdl`, saved in this repo
at `wsdl/tracelink-serialized-operations-manager.wsdl`. Read the WSDL for the exact
message shape before building a request; every operation name below is declared in it.

## Before you start

- Credentials come from the customer's TraceLink implementation team. There is no
  self-service signup and no OAuth — authentication is **HTTP Basic** (RFC 7617) over
  HTTPS. See `authentication/tracelink-authentication.yml`.
- Build and test against the **Validation** environment first:
  `https://itestapi.tracelink.com/soap/som`. It serves the same WSDL. See
  `sandbox/tracelink-sandbox.yml`.
- There is **no idempotency key** on any TraceLink surface
  (`conventions/tracelink-conventions.yml`). Do not blind-retry a write. On a 5xx,
  re-read state with a Get operation before retrying.

## Steps

1. **Verify the serial number.** Call `SerialNumberVerification` with the product
   identifier, serial number, lot, and expiry as declared in the WSDL. This is the
   DSCSA/VRS-style check: it answers whether the identifier is known and saleable.
2. **Read the entity.** Call `GetSerializedEntity` for the raw entity, or
   `GetSerializedEntityWithProduct` when you also need the product master attributes in
   the same response.
3. **Walk the hierarchy.** Call `GetSerialNumberHierarchy` to get the parent/child
   aggregation tree (item → case → pallet). If the operation returns a job reference
   rather than the tree, poll `FetchSerialNumberHierarchyResult` until it completes —
   SOM splits several reads into a launch/fetch pair rather than paginating.
4. **Check the lot.** Call `GetLotStatus` (poll `FetchLotStatusResult` where the launch/
   fetch pair applies) to confirm the lot is not on hold, recalled, or destroyed. A
   serial number can verify and still sit in a lot you must not ship.

## Reading errors

TraceLink does not use RFC 9457 problem details. Transport failures arrive as ordinary
HTTP status codes (`400 401 403 404 500 502 503 504`); business failures arrive as SOAP
faults and, on the OPUS JSON surfaces, as `header.isErr` plus `header.errCode`. Quote
`header.licensePlate` when raising a support ticket. See
`errors/tracelink-problem-types.yml`.

## Do not

- Do not construct an OpenAPI request. TraceLink publishes none; a REST-shaped call to
  `api.tracelink.com` returns `{"code":404,"message":"HTTP 404 Not Found"}`.
- Do not invent test serial numbers. TraceLink publishes no magic test values; use
  identifiers seeded in your own Validation tenant.
