# Agentic Credit Broking Protocol Standards and Conventions

## Status

This document records the standards and conventions adopted for the current specification set of the Agentic Credit Broking Protocol.

Its purpose is to establish a consistent baseline for the transport-neutral
technical specification, the schema appendix, and the REST and MCP bindings
before the detailed payload and operation definitions are filled out.

## 1. Role of This Document

This document defines:

- which external standards the document set SHOULD align with
- which conventions apply across the protocol specification set
- which conventions are transport-neutral
- which conventions are specific to the REST or MCP binding.

This document does not itself define the protocol semantics. It defines the
standards and editorial constraints within which the protocol will be
specified.

## 2. Normative Language

The specification set SHOULD use BCP 14 requirement language for
normative statements.

The key words `MUST`, `MUST NOT`, `REQUIRED`, `SHALL`, `SHALL NOT`, `SHOULD`,
`SHOULD NOT`, `RECOMMENDED`, `NOT RECOMMENDED`, `MAY`, and `OPTIONAL` SHOULD
be interpreted as described in BCP 14 when, and only when, they appear in all
capitals.

Sources:

- [RFC 2119](https://www.rfc-editor.org/info/rfc2119)
- [RFC 8174](https://www.rfc-editor.org/info/rfc8174)

## 3. Canonical Schema Standard

The shared transport-neutral schema set SHOULD use JSON Schema
Draft `2020-12`.

This SHOULD be the default schema dialect for the canonical protocol schemas
and for any machine-readable schema artefacts published as part of the release.

Rationale:

- it is current and well-supported
- it fits the transport-neutral schema appendix cleanly
- it aligns well with modern OpenAPI
- it aligns naturally with the current MCP specification's schema usage.

Source:

- [JSON Schema Draft 2020-12](https://json-schema.org/draft/2020-12)

## 4. REST Description Standard

The REST binding SHOULD use OpenAPI `3.1.0` as its description format.

OpenAPI is a binding-level description standard for the HTTP realisation of the
protocol. It is not the transport-neutral source of truth for protocol
semantics. The transport-neutral technical specification and canonical schema
set remain authoritative.

Rationale:

- OpenAPI `3.1` aligns with JSON Schema `2020-12`
- it is the most practical way to publish a formal REST binding
- it provides a standard structure for endpoints, payloads, and examples.

Source:

- [OpenAPI Specification 3.1.0](https://spec.openapis.org/oas/v3.1.0.html)

## 5. REST Error Standard

The REST binding SHOULD use HTTP Problem Details for structured error
responses.

The binding MAY define protocol-specific problem types, but it SHOULD NOT
invent an entirely separate error envelope for HTTP.

Source:

- [RFC 9457](https://www.rfc-editor.org/info/rfc9457)

## 6. MCP Versioning Standard

The MCP binding SHOULD target a specific dated MCP revision rather than an
unversioned or moving latest reference.

The current MCP binding targets the MCP specification revision dated
`2025-11-25`. Later bindings may target newer dated MCP revisions, but each
binding MUST identify the MCP revision it realises.

Rationale:

- MCP evolves actively
- version pinning avoids ambiguity
- a dated target makes the binding reviewable and reproducible.

Sources:

- [MCP Specification](https://modelcontextprotocol.io/specification/draft)
- [MCP 2025-11-25 Overview](https://modelcontextprotocol.io/specification/2025-11-25/basic)
- [MCP 2025-11-25 Server Features](https://modelcontextprotocol.io/specification/2025-11-25/server/index)

## 7. JSON Interoperability Profile

The protocol's canonical JSON payloads SHOULD stay within the I-JSON
interoperability profile wherever practical.

This means, at minimum, that the specification SHOULD prefer:

- UTF-8 encoded JSON
- unique object member names
- conservative numeric usage suitable for interoperable processing
- string-based representation for values that should not rely on lossy numeric
  handling.

Source:

- [RFC 7493](https://www.rfc-editor.org/info/rfc7493)

## 8. Identifier Standard

Opaque server-generated identifiers SHOULD use UUIDs.

For the current release, the preferred identifier family is UUID as standardised in
RFC `9562`. The exact UUID version to require across all identifier classes is
still open, but sortable UUIDs such as UUIDv7 are a strong candidate for
operational identifiers such as events and pending broker actions.

Source:

- [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562)

## 9. Timestamp Standard

Protocol payload timestamps SHOULD use RFC `3339` date-time strings in the
current release.

The current specification SHOULD avoid introducing richer timestamp suffixes or
time zone name extensions unless a concrete protocol need emerges that justifies
them. Simplicity is preferred for the first release.

Sources:

- [RFC 3339](https://www.rfc-editor.org/info/rfc3339)
- [RFC 9557](https://www.rfc-editor.org/info/rfc9557)

## 10. URI and Problem Type Conventions

Where the protocol uses URIs for type identifiers, problem identifiers, or
other namespaced references, those URIs SHOULD conform to the generic URI
syntax.

Source:

- [RFC 3986](https://www.rfc-editor.org/info/rfc3986)

## 11. Canonical JSON Field Naming

Wire-level JSON field names in the shared canonical schema set SHOULD use
`snake_case`.

This convention applies across:

- transport-neutral canonical schemas
- REST payloads derived from those schemas
- MCP tool input and output payload objects derived from those schemas.

The canonical schema set SHOULD NOT mix `snake_case` and `camelCase` field
names. Consistency takes precedence over local stylistic preferences of any one
binding.

Examples:

- `case_id`
- `pending_action`
- `event_type`
- `occurred_at`

## 12. Schema and Type Naming

Schema, object, and named model types in specification prose SHOULD use
`PascalCase`.

Examples:

- `CommandResponse`
- `PendingBrokerAction`
- `EntityReference`
- `FullStateResponse`

Enum wire values SHOULD use lowercase `snake_case` unless a specific external
standard requires a different form.

Examples:

- `information_request`
- `case_status_changed`
- `blocked_by_pending_action`

## 13. Identifier Field Naming

The specification SHOULD use a consistent identifier naming pattern in JSON
payloads:

- `id` for an object's own identifier
- `*_id` for references to other objects by identifier
- `type` for a wire-level discriminator or object kind where needed.

Examples:

- `id`
- `case_id`
- `action_id`
- `event_type`

## 14. Extension Model

The protocol SHOULD reserve an explicit extension point for future or
implementation-specific fields, but it SHOULD NOT use MCP's `_meta` field name
as the transport-neutral extension mechanism.

Rationale:

- `_meta` already has MCP-specific meaning
- a transport-neutral protocol SHOULD NOT make an MCP-specific reservation part
  of its canonical object model.

The exact extension field design remains open for the current release, but the
transport-neutral specification SHOULD preserve a clean separation between:

- protocol-defined fields
- transport-defined fields
- extension fields.

## 15. Examples and Normative Status

Examples in the document set SHOULD be treated as non-normative unless
explicitly stated otherwise.

The normative source of truth SHOULD remain:

- the transport-neutral technical specification for semantics
- the schema appendix and machine-readable canonical schemas for shared payload
  shape
- the binding documents for transport-specific mapping.

## 16. REST Resource Naming and URL Conventions

The REST binding SHOULD use these URL and resource naming conventions:

- path segments are lowercase
- multi-word resource names are hyphen-separated
- underscores are not used in URL path segments
- trailing slashes are not used
- resource names are nouns
- controller names are verbs or verb phrases.

This URL naming convention is separate from JSON payload field naming. REST
paths SHOULD therefore remain hyphenated even though payload fields use
`snake_case`.

Examples:

- `/pending-actions`
- `{case_root_url}/{case_id}/resolve-action`

## 17. REST Resource Archetype Vocabulary

The REST binding MAY use the resource archetype vocabulary as informal design guidance:

- `document`
- `collection`
- `store`
- `controller`

This vocabulary is binding-specific. It SHOULD NOT be treated as part of the
transport-neutral protocol semantics unless explicitly elevated later.

Source:

- [REST API URI Naming Conventions and Best Practices](https://restfulapi.net/resource-naming/)

## 18. Shared Schema Boundary

The following rule applies across the specification set:

- if a field changes protocol meaning, it belongs in the shared canonical
  schema set
- if a field exists only because of HTTP or MCP mechanics, it belongs in the
  relevant binding.

This rule governs the separation between:

- the transport-neutral technical specification
- the schema appendix
- the REST binding
- the MCP binding.

## 19. Relationship to Companion Documents

- [protocol-goals-and-scope.md](protocol-goals-and-scope.md)
  defines the release intent and scope.
- [protocol-technical-spec.md](protocol-technical-spec.md)
  defines the transport-neutral semantics.
- [protocol-schema-appendix.md](protocol-schema-appendix.md)
  defines the shared schema catalogue.
- [protocol-rest-binding.md](protocol-rest-binding.md)
  defines the REST realisation.
- [protocol-mcp-binding.md](protocol-mcp-binding.md)
  defines the MCP realisation.
