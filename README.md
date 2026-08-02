# TraceLink

TraceLink, Inc. is a supply chain digitalization company for the life sciences and
healthcare industries, best known for pharmaceutical serialization and track-and-trace
compliance (US DSCSA, EU FMD, and roughly two dozen national regimes) delivered on its
multienterprise OPUS network platform.

- Website: https://www.tracelink.com/
- Documentation: https://opus.tracelink.com/documentation/
- GitHub: https://github.com/tracelink

## API surfaces

TraceLink publishes **no OpenAPI**. Its machine-readable contracts are WSDL plus canonical
JSON Schema, across five distinct surfaces:

| Surface | Style | Contract |
|---|---|---|
| Serialized Operations Manager | SOAP 1.1 | `wsdl/tracelink-serialized-operations-manager.wsdl` (40 operations) |
| Product Track | SOAP 1.1 | `wsdl/tracelink-product-track.wsdl` (2 operations) |
| Serial Number Exchange | SOAP 1.1 | `wsdl/tracelink-serial-number-exchange.wsdl` (1 operation) |
| Smart Event Manager | synchronous JSON/REST | published PDF API guide |
| OPUS Platform | single-endpoint event RPC (`POST /api/events`) + GraphQL | published event-name catalog |

Asynchronous B2B message exchange runs alongside these over AS2, SFTP, and HTTP POST
using EPCIS, EDI ANSI X12, SAP IDoc, and TraceLink XML — see
`asyncapi/tracelink-event-surface.yml`.

## Artifacts in this repo

- `wsdl/` — three WSDLs fetched verbatim from `https://api.tracelink.com/soap/*?wsdl`
- `json-schema/` — 37 canonical object schemas (JSON Schema draft-04) harvested from
  TraceLink's public `code-samples` repository
- `well-known/` — RFC 9116 `security.txt` and the discovery probe index
- `llms/` — TraceLink's published `llms.txt`
- `authentication/`, `conventions/`, `errors/`, `conformance/`, `lifecycle/`,
  `changelog/`, `sandbox/`, `data-model/` — captured semantics
- `security/` — domain security probe, vulnerability disclosure, trust center
- `mcp/`, `skills/` — API Evangelist-authored agent artifacts (marked as such in their
  frontmatter; TraceLink publishes neither)
