# Agentic Credit Broking Protocol REST Binding

## Status

This document defines the REST binding for the protocol.

It is a transport-specific companion to [protocol-technical-spec.md](protocol-technical-spec.md). If there is any conflict between this document and the transport-neutral semantics, the transport-neutral specification should take precedence unless and until a later revision explicitly states otherwise.

This binding SHOULD also follow the shared standards and conventions recorded in [protocol-standards-and-conventions.md](protocol-standards-and-conventions.md).

## 1. Purpose

This binding maps the transport-neutral protocol semantics onto HTTP resources, methods, request bodies, responses, and error signalling conventions.

The purpose of the REST binding is not to redefine the protocol. It is to make the protocol usable through a conventional web API shape while preserving the same underlying operation and lifecycle semantics.

## 2. Binding Principles

This binding SHOULD preserve:

- operation equivalence with the transport-neutral specification
- event and pending-action semantics
- entity and identifier semantics
- lifecycle and gate behaviour
- materially equivalent error meaning.

This binding MAY define HTTP-specific details such as:

- endpoint structure
- HTTP methods
- status code mapping
- media type assumptions
- request and response serialisation conventions.

## 3. Resource and Endpoint Model

The REST binding includes three kinds of HTTP surface:

- case-oriented protocol endpoints
- a read-only HTTP resource surface for broker-published vocabularies
- an explicit discovery and instruction layer for AI-capable clients.

At minimum, a conforming REST implementation MUST expose HTTP-accessible resources corresponding to the mandatory broker-published resources defined in the transport-neutral specification:

- `financial_fact_types`
- `party_attribute_types`
- `goal_types`
- `financial_product_features`
- `financial_product_categories`
- `evidence_types`

These resources SHOULD be readable without first opening a case, because a User Agent may need them before it can classify user input correctly. The REST binding exposes them through a stable, read-only vocabulary namespace in addition to the command and state endpoints.

### 3.1 Discovery Entry Point

A conforming REST implementation SHOULD expose a single combined discovery document at:

- `/.well-known/agentic-credit`

This document is the REST bootstrap artefact for AI-capable clients. Its role is to tell the client:

- which protocol version it is speaking
- where the behavioural instructions live
- where the OpenAPI description lives
- where case URLs are rooted
- where the mandatory broker-published resources live.

The use of `/.well-known/` is deliberate. It gives the REST binding one stable location that a generic client can try first without knowing the broker's endpoint layout in advance. The `agentic-credit` suffix is not currently IANA-registered; production registration remains a follow-up action, and implementations MAY use an alternative non-well-known bootstrap path until registration is complete.

### 3.2 Combined Discovery Document Model

The REST binding uses a single combined discovery document rather than split discovery and instruction documents.

That document SHOULD contain at least:

- `protocol_version`
- `title`
- `summary`
- `instructions_markdown`
- `openapi_url`
- `case_root_url`
- `resource_root_url`
- `resources`.

Field expectations:

- `protocol_version`: broker-declared protocol version string
- `title`: short human-readable title for the broker's REST protocol surface
- `summary`: short natural-language summary of the broker surface
- `instructions_markdown`: behavioural guidance for AI-capable clients using the protocol through REST
- `openapi_url`: canonical URL for the OpenAPI description of the REST binding
- `case_root_url`: canonical URL for the REST case collection root
- `resource_root_url`: canonical URL for the published-resource root
- `resources`: object mapping each mandatory published resource kind to a concrete HTTP URL.

The `resources` object SHOULD contain links for:

- `financial_fact_types`
- `party_attribute_types`
- `goal_types`
- `financial_product_features`
- `financial_product_categories`
- `evidence_types`.

### 3.3 Instruction Content Model

The behavioural content published in `instructions_markdown` SHOULD cover, at minimum:

- the role of the client as User Agent
- the meaning of the six canonical operations
- how to treat `pending_action` as the active gate after a command
- when `Provide` may satisfy an `information_request`
- how to treat regulated content
- how to capture explicit decisions
- when to call `Get State`
- how to use broker-published vocabularies when forming structured payloads.

The discovery document is not a substitute for OpenAPI. OpenAPI remains the authoritative REST transport description for endpoint paths, methods, and request and response bodies. The discovery document provides the behavioural and semantic guidance that an AI-capable client needs in addition to raw endpoint mechanics.

The discovery document SHOULD give the client enough information to construct or recognise the canonical case and resource URLs without assuming that the broker uses `/cases` or `/resources` specifically.

### 3.4 Expected Client Bootstrap Flow

The expected bootstrap flow for an AI-capable REST client is:

1. Fetch `/.well-known/agentic-credit`.
2. Read `instructions_markdown`.
3. Read `case_root_url` and `resource_root_url`.
4. Fetch the referenced OpenAPI description.
5. Fetch the referenced broker-published resources.
6. Begin invoking the REST operations.

### 3.5 Endpoint Layout

For this release, the REST binding uses a hybrid endpoint model:

- cases are addressed through a broker-declared case collection root
- full state retrieval is exposed as a case resource read
- mutating protocol operations other than case creation are exposed as operation endpoints under a case
- broker-published vocabularies are exposed as read-only resources under a broker-declared resource root.

The recommended endpoint shape is:

- `POST {case_root_url}`
- `GET {case_root_url}/{case_id}`
- `POST {case_root_url}/{case_id}/provide`
- `POST {case_root_url}/{case_id}/select`
- `POST {case_root_url}/{case_id}/resolve-action`
- `POST {case_root_url}/{case_id}/withdraw`
- `GET {resource_root_url}/financial-fact-types`
- `GET {resource_root_url}/party-attribute-types`
- `GET {resource_root_url}/goal-types`
- `GET {resource_root_url}/financial-product-features`
- `GET {resource_root_url}/financial-product-categories`
- `GET {resource_root_url}/evidence-types`.

The precise root paths are a broker choice and are discovered through `case_root_url`, `resource_root_url`, and the OpenAPI description. This hybrid model keeps case retrieval idiomatic for REST while still making the command semantics of `provide`, `select`, `resolve_action`, and `withdraw` explicit rather than pretending they are generic resource updates.

`Get State` intentionally maps to `GET {case_root_url}/{case_id}` without an operation-name path segment because it is a read of the case resource projection rather than a command endpoint.

### 3.6 Resource Addressing and Caching

Published broker resources SHOULD be cacheable over HTTP.

At minimum, the REST binding SHOULD support ordinary HTTP cache validation mechanisms such as:

- `ETag`
- `Last-Modified`
- conditional `GET`.

Case-state responses SHOULD be treated more conservatively. `GET {case_root_url}/{case_id}` MAY support freshness metadata, but clients SHOULD assume case state is volatile and SHOULD re-fetch when they need the broker's current projection.

## 4. Operation Mapping

This section maps each canonical protocol operation to a REST endpoint pattern rooted at the broker-declared `case_root_url`.

### 4.1 Open Case

`Open Case` maps to:

- endpoint pattern: `POST {case_root_url}`
- request body: `OpenCaseRequest`
- success response shape: `CommandResponse`
- expected success status: `201 Created`.

On success, the response SHOULD include:

- a `Location` header pointing to `{case_root_url}/{case_id}`
- a response body containing the `CommandResponse`.

### 4.2 Provide

`Provide` maps to:

- endpoint pattern: `POST {case_root_url}/{case_id}/provide`
- request body: `ProvideRequest`
- success response shape: `CommandResponse`
- expected success status: `200 OK`.

### 4.3 Get State

`Get State` maps to:

- endpoint pattern: `GET {case_root_url}/{case_id}`
- success response shape: `FullStateResponse`
- expected success status: `200 OK`.

Caching assumptions:

- clients SHOULD treat the representation as a current projection, not a durable historical snapshot
- ordinary HTTP freshness metadata MAY be present
- clients SHOULD re-fetch when they need authoritative current state after intervening commands.

### 4.4 Select

`Select` maps to:

- endpoint pattern: `POST {case_root_url}/{case_id}/select`
- request body: `SelectRequest`
- success response shape: `CommandResponse`
- expected success status: `200 OK`.

### 4.5 Resolve Action

`Resolve Action` maps to:

- endpoint pattern: `POST {case_root_url}/{case_id}/resolve-action`
- request body: `ResolveActionRequest`
- success response shape: `CommandResponse`
- expected success status: `200 OK`.

### 4.6 Withdraw

`Withdraw` maps to:

- endpoint pattern: `POST {case_root_url}/{case_id}/withdraw`
- request body: `WithdrawRequest`
- success response shape: `CommandResponse`
- expected success status: `200 OK`.

## 5. Serialisation Model

This section defines the JSON serialisation model for the REST binding.

The REST binding uses JSON representations derived directly from the transport-neutral schemas.

The REST serialisation rules are:

- payload bodies use JSON
- field names use `snake_case`
- timestamps use RFC `3339` date-time strings
- identifiers are serialised as opaque strings
- enums and controlled values use lowercase `snake_case`
- entity references are serialised as compact objects containing at least `id` and `entity_type`
- inline entities are serialised as typed objects containing at least `id`, `entity_type`, and type-specific fields.

For path-scoped mutating operations (`provide`, `select`, `resolve_action`, `withdraw`), request bodies MAY include `case_id` for transport parity with the canonical schema family. When present, that body `case_id` MUST match the path `case_id`; a mismatch MUST be rejected as `malformed_request`.

The REST binding SHOULD avoid introducing HTTP-specific wrapper fields that do not change protocol meaning. In particular:

- `CommandResponse` SHOULD be returned directly as the response body for successful mutating operations
- `FullStateResponse` SHOULD be returned directly as the response body for `GET {case_root_url}/{case_id}`
- published broker resources SHOULD be returned directly as their canonical `PublishedResource` payloads.

Where request or response metadata must be expressed through HTTP rather than JSON, such as `Location`, `ETag`, or cache headers, that metadata is additive and MUST NOT alter the meaning of the JSON payload.

## 6. HTTP Status and Error Mapping

This section maps transport-neutral errors onto HTTP semantics.

The REST binding SHOULD use HTTP Problem Details for structured error responses.

The recommended mapping is:

- `malformed_request` -> `400 Bad Request`
- `unauthenticated` -> `401 Unauthorized`
- `unauthorised` -> `403 Forbidden`, or `404 Not Found` where the broker deliberately avoids disclosing protected case existence
- `unauthorized` -> compatibility alias of `unauthorised`, mapped with the same HTTP handling
- `invalid_reference` -> `404 Not Found` when the missing reference is the case addressed by `{case_id}` in the URL path, otherwise `400 Bad Request` for non-primary references such as `action_id` or selected entity references
- `invalid_operation_for_state` -> `409 Conflict`
- `blocked_by_pending_action` -> `409 Conflict`
- `trust_level_insufficient` -> `403 Forbidden`
- `unknown_or_unsupported_value` -> `422 Unprocessable Content`
- `content_integrity_failure` -> `400 Bad Request`
- `stale_or_superseded_action` -> `409 Conflict`
- `concurrency_or_sequence_conflict` -> `409 Conflict`
- `internal_processing_failure` -> `500 Internal Server Error`.

Every REST error response SHOULD carry a problem-details body that preserves the transport-neutral error semantics. At minimum that body SHOULD allow the client to recover at least:

- the protocol error category
- a short human-readable message
- the relevant `case_id` when applicable
- the relevant `action_id` when the request is blocked by a pending action
- the offending field or identifier when practical
- the required scope, trust capability, or integrity subject when that can be stated safely.

The HTTP status code alone is not sufficient. Clients must be able to distinguish, for example, a blocked request from an invalid lifecycle transition even when both use `409 Conflict`.

For `unauthenticated`, the broker SHOULD use `WWW-Authenticate` where applicable so that the client can initiate or refresh authentication. For `unauthorised`, the broker SHOULD avoid revealing whether a case exists when the authenticated caller lacks access to it. In that situation, a `404 Not Found` response MAY be used as the HTTP non-disclosure pattern while the broker records the underlying authorisation failure internally. Implementations MAY accept `unauthorized` as an inbound or emitted compatibility alias, but `unauthorised` remains the canonical protocol spelling.

## 7. Concurrency and Idempotency

This section defines the REST position on request replay and sequencing.

For this release:

- `GET {case_root_url}/{case_id}` and `GET {resource_root_url}/...` are naturally idempotent
- `POST {case_root_url}` is not assumed idempotent unless a broker explicitly adds idempotency-key support
- `POST {case_root_url}/{case_id}/provide`, `POST {case_root_url}/{case_id}/select`, `POST {case_root_url}/{case_id}/resolve-action`, and `POST {case_root_url}/{case_id}/withdraw` are not assumed idempotent by default.

The current specification does not require an idempotency-key mechanism, but a broker MAY add one. If such a mechanism is added, it SHOULD be documented explicitly in the OpenAPI description and SHOULD preserve the transport-neutral semantics of the operation.

This release does not require optimistic concurrency headers such as `If-Match`, nor does it require version-precondition checks on mutating requests. However, a broker MAY expose case-local sequencing metadata in responses.

If the REST binding exposes sequence markers:

- they SHOULD appear in the response body using the transport-neutral fields such as `sequence` or `last_sequence`
- the broker MAY also mirror them into HTTP headers for convenience
- clients MUST treat them as broker ordering hints unless and until a later version defines stronger concurrency semantics.

## 8. Authentication and Security

This section defines the REST binding's initial realisation of the transport-neutral security model in [protocol-technical-spec.md](protocol-technical-spec.md#17-security-model).

### 8.1 HTTPS and Protected Case Surface

Production REST implementations MUST serve the protocol surface over HTTPS.

The broker MUST authenticate and authorise requests before allowing access to case-scoped endpoints that expose private case data, create or mutate case state, resolve broker actions, or retrieve case state. This includes `POST {case_root_url}` where the call creates a private case that will later be continued by the same user authority.

Discovery documents, OpenAPI descriptions, broker instructions, and broker-published vocabulary resources MAY be public or broadly readable. Public access to those resources MUST NOT imply access to any case-scoped endpoint.

### 8.2 OAuth-Based Authorisation

For production internet-facing REST deployments, the RECOMMENDED authorisation profile is OAuth 2.1 or OpenID Connect over HTTPS.

The broker MAY use JWT access tokens, opaque access tokens with server-side introspection, or another OAuth-compatible token representation. Regardless of token format, the broker MUST verify or resolve, at minimum:

- issuer or issuing authority
- intended audience or protected resource
- expiration
- scopes, permissions, or equivalent operation capability
- authenticated protocol participant or client identity
- account-bound user authority or guest case-continuation authority.

For account-bound cases, the token subject or resolved token state identifies a broker-recognised user account or equivalent durable user authority.

For guest cases, the token subject or resolved token state identifies a pseudonymous guest participant, guest session, or guest case-continuation authority. A guest OAuth token MUST NOT be treated as proof that the broker knows the user's durable real-world identity.

The broker MAY bind case access either by carrying case-specific authorisation in the token or by resolving the token subject to a server-side access-control record. In either model, `case_id` remains an identifier only and MUST NOT authorise access by itself.

Access tokens, refresh tokens, guest continuation credentials, and other secrets MUST NOT be carried in protocol JSON request bodies, protocol JSON response bodies, transcripts, broker-authored content, or user-facing prose. They belong in HTTP authentication headers, secure cookies, or another binding-defined credential channel.

OpenAPI descriptions SHOULD declare the security schemes and scopes required for each protected endpoint. Operation-specific scopes or permissions SHOULD be narrow enough to distinguish reading case state, providing data, making selections, resolving broker actions, withdrawing, and reading public resources.

### 8.3 Case Access Control and Guest Continuation

The broker MUST apply the same case access-control decision across all endpoints that operate on the same case. A caller that cannot access `GET {case_root_url}/{case_id}` MUST NOT be able to infer or mutate the same case through `POST {case_root_url}/{case_id}/provide`, `POST {case_root_url}/{case_id}/select`, `POST {case_root_url}/{case_id}/resolve-action`, or `POST {case_root_url}/{case_id}/withdraw`.

Guest continuation credentials SHOULD be short-lived, case-scoped, revocable, and bound to the authenticated client or user-agent context where practical. A broker SHOULD require re-authentication, token refresh, account binding, identity verification, or broker-controlled surface completion before allowing a guest case to perform higher-risk actions.

### 8.4 Trust-Calibrated Capability

REST authentication and OAuth scopes establish authority to call endpoints. They do not by themselves establish that the User Agent is trusted to handle every regulated moment directly.

The broker MUST still apply trust-calibrated capability policy when processing `Select`, `Resolve Action`, and any operation that could cause or complete a regulated pending broker action. If the authenticated participant lacks the required trust capability, the broker SHOULD direct the user to the broker-controlled surface where possible. If the participant attempts an operation it is not trusted to perform directly, the broker SHOULD return `trust_level_insufficient`.

### 8.5 Regulated Content Integrity

For this release, REST protects broker-authored regulated content in transit through HTTPS and preserves auditability through stable action identifiers and broker evidence records. A broker SHOULD record the exact regulated content issued for each pending broker action, together with its action identifier, timestamp, and resolution outcome.

A broker MAY add stronger regulated-content integrity controls, such as content digests, detached signatures, signed response envelopes, or notarisation mechanisms. If such controls are used, they SHOULD be documented in the OpenAPI description and failures SHOULD map to `content_integrity_failure`.

The REST binding does not yet mandate one regulated-content signing or notarisation mechanism. That remains a production-profile question for later versions.

## 9. Worked REST Examples

This section mirrors one coherent case flow:

- the user opens a case for debt consolidation
- the broker raises an `information_request` asking for missing financial facts
- the user satisfies that gate with `Provide`
- the broker produces a plan
- the user selects the plan
- the broker raises a regulated `disclosure`
- the user acknowledges the disclosure through `Resolve Action`
- the User Agent retrieves the full current case projection with `Get State`.

The identifiers, timestamps, and broker-authored content below are illustrative. The examples are intended to show shape and sequencing, not to define canonical business copy. Authentication and authorisation material is omitted from the examples.

For illustration only, these examples use `/cases` as the discovered `case_root_url`.

### 9.1 `Open Case`

Request:

```http
POST /cases HTTP/1.1
Content-Type: application/json

{
  "initial_goals": [
    {
      "id": "goal_001",
      "entity_type": "financial_goal",
      "goal_type_id": "debt_consolidation",
      "goal_statement": "Reduce unsecured debt into one manageable repayment.",
      "target_timeframe": "12_months"
    }
  ],
  "initial_party_attributes": [
    {
      "id": "party_attr_001",
      "entity_type": "party_attribute",
      "attribute_type_id": "employment_status",
      "value": "full_time"
    }
  ],
  "transcript": "The user wants help consolidating existing unsecured debt."
}
```

Response:

```http
HTTP/1.1 201 Created
Location: /cases/case_001
Content-Type: application/json

{
  "case_id": "case_001",
  "events": [
    {
      "id": "evt_001",
      "event_type": "goals_updated",
      "case_id": "case_001",
      "occurred_at": "2026-04-23T10:00:00Z",
      "sequence": 1,
      "entity_refs": [
        {
          "id": "goal_001",
          "entity_type": "financial_goal"
        }
      ],
      "entities": [
        {
          "id": "goal_001",
          "entity_type": "financial_goal",
          "goal_type_id": "debt_consolidation",
          "goal_statement": "Reduce unsecured debt into one manageable repayment.",
          "target_timeframe": "12_months"
        }
      ]
    }
  ],
  "pending_action": {
    "id": "action_001",
    "action_type": "information_request",
    "case_id": "case_001",
    "issued_at": "2026-04-23T10:00:00Z",
    "blocking": true,
    "regulated": false,
    "content": "Please provide your net monthly income and total unsecured debt balance so I can assess suitable consolidation options.",
    "content_format": "text/markdown",
    "response_expectation": "provide_information",
    "subjects": [
      {
        "entity_ref": {
          "id": "goal_001",
          "entity_type": "financial_goal"
        },
        "subject_role": "primary"
      }
    ],
    "information_scope": {
      "needs": [
        {
          "item_type": "income_net_monthly",
          "information_context": "current_profile"
        },
        {
          "item_type": "debt_unsecured_total_balance",
          "information_context": "current_profile",
          "target_entity_ref": {
            "id": "goal_001",
            "entity_type": "financial_goal"
          }
        }
      ]
    }
  },
  "request_id": "req_001",
  "sequence": 1,
  "generated_at": "2026-04-23T10:00:00Z"
}
```

This shows the normal opening pattern where the broker accepts an initial goal but immediately raises a blocking `information_request`. The gate is visible in `pending_action`, not hidden in out-of-band prose.

### 9.2 `Provide` to Satisfy the `information_request`

Request:

```http
POST /cases/case_001/provide HTTP/1.1
Content-Type: application/json

{
  "case_id": "case_001",
  "facts": [
    {
      "id": "fact_001",
      "entity_type": "financial_fact",
      "fact_type_id": "income_net_monthly",
      "value": {
        "amount": "2800",
        "currency": "GBP"
      },
      "operator": "equal_to",
      "information_context": "current_profile"
    },
    {
      "id": "fact_002",
      "entity_type": "financial_fact",
      "fact_type_id": "debt_unsecured_total_balance",
      "value": {
        "amount": "14500",
        "currency": "GBP"
      },
      "operator": "equal_to",
      "information_context": "current_profile"
    }
  ],
  "transcript": "The user confirmed take-home pay of 2,800 GBP and total unsecured debt of 14,500 GBP."
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "case_id": "case_001",
  "events": [
    {
      "id": "evt_002",
      "event_type": "profile_updated",
      "case_id": "case_001",
      "occurred_at": "2026-04-23T10:02:00Z",
      "sequence": 2,
      "entity_refs": [
        {
          "id": "fact_001",
          "entity_type": "financial_fact"
        },
        {
          "id": "fact_002",
          "entity_type": "financial_fact"
        }
      ],
      "entities": [
        {
          "id": "fact_001",
          "entity_type": "financial_fact",
          "fact_type_id": "income_net_monthly",
          "value": {
            "amount": "2800",
            "currency": "GBP"
          },
          "operator": "equal_to",
          "information_context": "current_profile"
        },
        {
          "id": "fact_002",
          "entity_type": "financial_fact",
          "fact_type_id": "debt_unsecured_total_balance",
          "value": {
            "amount": "14500",
            "currency": "GBP"
          },
          "operator": "equal_to",
          "information_context": "current_profile"
        }
      ]
    },
    {
      "id": "evt_003",
      "event_type": "plans_ready",
      "case_id": "case_001",
      "occurred_at": "2026-04-23T10:02:01Z",
      "sequence": 3,
      "entity_refs": [
        {
          "id": "plan_001",
          "entity_type": "financial_plan"
        }
      ],
      "entities": [
        {
          "id": "plan_001",
          "entity_type": "financial_plan",
          "steps": [
            "Consolidate unsecured balances into a single fixed-term personal loan.",
            "Keep monthly repayment below 450 GBP."
          ]
        }
      ]
    }
  ],
  "pending_action": null,
  "request_id": "req_002",
  "sequence": 3,
  "generated_at": "2026-04-23T10:02:01Z"
}
```

This is the main gate exception in practice: `Provide` is accepted because the active blocking action is an `information_request`. The same call would normally be rejected if the active gate were a `disclosure` or `consent`.

### 9.3 `Select` Producing a Regulated Pending Action

Request:

```http
POST /cases/case_001/select HTTP/1.1
Content-Type: application/json

{
  "case_id": "case_001",
  "selection": {
    "entity_ref": {
      "id": "plan_001",
      "entity_type": "financial_plan"
    },
    "transcript": "The user wants to proceed with the consolidation plan."
  }
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "case_id": "case_001",
  "events": [],
  "pending_action": {
    "id": "action_002",
    "action_type": "disclosure",
    "case_id": "case_001",
    "issued_at": "2026-04-23T10:03:00Z",
    "blocking": true,
    "regulated": true,
    "content": "Important: debt consolidation may extend your repayment period and increase the total amount repaid. Please review this before continuing.",
    "content_format": "text/markdown",
    "response_expectation": "acknowledge",
    "subjects": [
      {
        "entity_ref": {
          "id": "plan_001",
          "entity_type": "financial_plan"
        },
        "subject_role": "selected_option"
      }
    ]
  },
  "request_id": "req_003",
  "sequence": 4,
  "generated_at": "2026-04-23T10:03:00Z"
}
```

This example shows a different kind of gate. After the plan is selected, the broker pauses progression behind a regulated `disclosure`. The User Agent must preserve the broker-authored content faithfully and cannot bypass it with another `Select`.

### 9.4 `Resolve Action` for the Disclosure

Request:

```http
POST /cases/case_001/resolve-action HTTP/1.1
Content-Type: application/json

{
  "case_id": "case_001",
  "action_id": "action_002",
  "resolution": {
    "outcome": "acknowledged",
    "responded_at": "2026-04-23T10:04:00Z",
    "transcript": "The user confirms they have read and understood the disclosure."
  }
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "case_id": "case_001",
  "events": [
    {
      "id": "evt_004",
      "event_type": "offers_ready",
      "case_id": "case_001",
      "occurred_at": "2026-04-23T10:04:01Z",
      "sequence": 5,
      "entity_refs": [
        {
          "id": "offer_001",
          "entity_type": "product_offer"
        }
      ],
      "entities": [
        {
          "id": "offer_001",
          "entity_type": "product_offer",
          "product_category_id": "personal_loan",
          "feature_values": {
            "apr_representative": "12.9",
            "term_months": 48,
            "monthly_payment": {
              "amount": "389",
              "currency": "GBP"
            }
          }
        }
      ]
    }
  ],
  "pending_action": null,
  "request_id": "req_004",
  "sequence": 5,
  "generated_at": "2026-04-23T10:04:01Z"
}
```

### 9.5 `Get State`

Request:

```http
GET /cases/case_001 HTTP/1.1
Accept: application/json
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "case_id": "case_001",
  "case_status": "open",
  "entities": [
    {
      "id": "goal_001",
      "entity_type": "financial_goal",
      "goal_type_id": "debt_consolidation",
      "goal_statement": "Reduce unsecured debt into one manageable repayment.",
      "target_timeframe": "12_months"
    },
    {
      "id": "plan_001",
      "entity_type": "financial_plan",
      "steps": [
        "Consolidate unsecured balances into a single fixed-term personal loan.",
        "Keep monthly repayment below 450 GBP."
      ]
    },
    {
      "id": "offer_001",
      "entity_type": "product_offer",
      "product_category_id": "personal_loan",
      "feature_values": {
        "apr_representative": "12.9",
        "term_months": 48,
        "monthly_payment": {
          "amount": "389",
          "currency": "GBP"
        }
      }
    }
  ],
  "pending_action": null,
  "last_sequence": 5,
  "generated_at": "2026-04-23T10:04:02Z",
  "resource_versions": {
    "financial_fact_types": "2026-04-23",
    "party_attribute_types": "2026-04-23",
    "goal_types": "2026-04-23",
    "financial_product_categories": "2026-04-23",
    "financial_product_features": "2026-04-23",
    "evidence_types": "2026-04-23"
  }
}
```

This final read shows the shape difference between `CommandResponse` and `FullStateResponse`. The command responses only reported deltas plus the active gate. `Get State` returns the broker's current case projection.

## 10. Open REST Questions

The remaining REST-specific questions for later refinement are:

- whether the hybrid endpoint model should remain normative in later revisions or collapse toward a more purely operation-oriented or resource-oriented style
- whether idempotency keys should become recommended or required for selected mutating operations
- whether any asynchronous event feed, webhook, or server-push mechanism belongs in a later REST binding
- how strongly the structure and wording of `instructions_markdown` should be standardised
- whether later versions should split the combined discovery document into separate discovery and instruction artefacts.
