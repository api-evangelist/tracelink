---
name: Commission, aggregate, and ship serialized product through TraceLink
description: >-
  Walk a serialized batch through TraceLink Serialized Operations Manager — commission
  the serial numbers, aggregate them into cases and pallets, record the shipment against
  an order, and confirm the receipt.
api: wsdl/tracelink-serialized-operations-manager.wsdl
surface: SOAP 1.1 over HTTPS
endpoint: https://api.tracelink.com/soap/som
operations:
  - serialNumbersRequest
  - Commission
  - Aggregate
  - AggregateUpdate
  - Disaggregate
  - UpdateOrderShipment
  - UpdateOrderShipmentAsynchronously
  - LaunchAsyncUpdateOrderShipment
  - UpdateOrderReceipt
  - FetchUpdateOrderReceiptResult
  - ListOpenOrdersAndReceipts
  - VoidShipment
generated: '2026-08-02'
method: generated
x-authored-by: API Evangelist enrichment pipeline
source: wsdl/tracelink-serialized-operations-manager.wsdl, wsdl/tracelink-serial-number-exchange.wsdl
---

# Record a serialized shipment

This is the core track-and-trace loop. Two SOAP services are involved: Serial Number
Exchange issues the numbers, Serialized Operations Manager records what happens to them.
Both WSDLs are in `wsdl/` and both are served publicly at `?wsdl` on
`api.tracelink.com`.

## Steps

1. **Get serial numbers** — `serialNumbersRequest` on
   `https://api.tracelink.com/soap/snx/snrequest` (service `SerialNumberRequestService`).
   This returns the range your packaging line will apply.
2. **Commission** — `Commission` on SOM. This is the moment a serial number becomes a
   real serialized entity in TraceLink. Confirm with `GetSerializedEntity`.
3. **Aggregate** — `Aggregate` to build the item → case → pallet hierarchy.
   `AggregateUpdate` amends an existing parent; `Disaggregate` removes children;
   `ResetAggregation` clears the tree for a parent. Verify with
   `GetSerialNumberHierarchy`.
4. **Ship** — `UpdateOrderShipment` for a synchronous shipment. For a large shipment use
   the asynchronous pair: `LaunchAsyncUpdateOrderShipment` (or
   `UpdateOrderShipmentAsynchronously`) to start the job, then poll the matching fetch
   operation for the result. TraceLink models bulk work as launch-then-poll rather than
   paginating.
5. **Receive** — the receiving party calls `UpdateOrderReceipt` and polls
   `FetchUpdateOrderReceiptResult`. Use `ListOpenOrdersAndReceipts` to find what is
   outstanding, and `GetReceipt` / `GetOrder` to read one.
6. **Correct** — `VoidShipment` reverses a shipment; `ChangeDeliveryNumber` and
   `FindDeliveryNumbers` fix and locate delivery references. `Destroy` and `Decommission`
   remove product from circulation; `TakeSample` records a sample pull.

## Compliance side effects

These calls are not neutral. A shipment recorded here can trigger regulatory reporting —
the NMVS Compliance application decides when a transaction must be reported to a national
medicines verification system, and Product Track carries US DSCSA obligations. Treat
every write as a regulated business event, not a database update. See
`conformance/tracelink-conformance.yml`.

## Async, errors, and retries

- Long operations return a job handle; poll the paired `Fetch*` operation.
- HTTP statuses in play: `200 307 400 401 403 404 500 502 503 504`
  (`errors/tracelink-problem-types.yml`).
- No idempotency key exists. Before retrying a failed `Commission`, `Aggregate`, or
  `UpdateOrderShipment`, read current state — a duplicate commission is a data-quality
  incident, not a no-op.

## Environments

Validation: `https://itestapi.tracelink.com/soap/som` (same WSDL). Production:
`https://api.tracelink.com/soap/som`. Credentials are HTTP Basic and are issued per
environment by TraceLink.
