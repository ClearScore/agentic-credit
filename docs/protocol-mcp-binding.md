# Agentic Credit Broking Protocol MCP Binding

## Status

This document defines the MCP binding for the protocol. It targets the MCP specification revision dated `2025-11-25` for tool, resource, prompt, and authorisation semantics. That target was current when this release set was authored; later MCP revisions may be targeted by a future binding revision.

It is a transport-specific companion to [protocol-technical-spec.md](protocol-technical-spec.md). If there is any conflict between this document and the transport-neutral semantics, the transport-neutral specification should take precedence unless and until a later revision explicitly states otherwise.

This binding SHOULD also follow the shared standards and conventions recorded in [protocol-standards-and-conventions.md](protocol-standards-and-conventions.md).

## 1. Purpose

This binding maps the transport-neutral protocol semantics onto an MCP tool surface.

The purpose of the MCP binding is to describe how a User Agent that operates as an MCP client can invoke the protocol, receive structured results, handle pending broker actions, and retrieve case state while preserving the same core semantics defined in the transport-neutral specification.

## 2. Binding Principles

This binding SHOULD preserve:

- operation equivalence with the transport-neutral specification
- event and pending-action semantics
- entity and identifier semantics
- lifecycle and gate behaviour
- materially equivalent error meaning.

This binding MAY define MCP-specific details such as:

- tool names
- tool descriptions
- tool input and output schemas
- how state is surfaced through tools
- how MCP-specific affordances are used or intentionally not used in this release.

## 3. Tool Surface Model

The MCP surface is not tools-only. It consists of:

- the canonical protocol tool set
- a required set of read-only MCP resources for broker-published vocabularies
- a broker instruction layer delivered through MCP-native affordances.

At minimum, a conforming MCP server MUST expose MCP resources corresponding to the mandatory broker-published resources defined in the transport-neutral specification:

- `financial_fact_types`
- `party_attribute_types`
- `goal_types`
- `financial_product_features`
- `financial_product_categories`
- `evidence_types`

These resources MUST be retrievable without an existing case and SHOULD be available to a client at connection time or through normal MCP resource discovery. The precise resource URIs and discovery mechanics are binding-specific, but the semantic payloads SHOULD align with the shared canonical schemas described by the transport-neutral specification and schema appendix.

### 3.1 Broker Instructions in MCP

MCP is the preferred transport for dynamic AI-capable protocol participation because MCP already has native affordances for delivering tool descriptions, prompts, and resources to a model-driven client.

A conforming MCP server SHOULD expose broker behavioural instructions through:

- one primary broker instruction prompt
- one mirrored read-only instruction resource.

The prompt is the primary delivery mechanism for AI-capable clients. The mirrored resource exists for inspection, caching, and non-prompt-oriented MCP clients.

### 3.2 Shared Behavioural Content Across Transports

The behavioural content delivered through MCP SHOULD be materially the same as the behavioural content delivered through the REST discovery document's `instructions_markdown`.

That shared behavioural content SHOULD cover, at minimum:

- the role of the client as User Agent
- the meaning of the six canonical operations
- how to treat `pending_action` as the active gate after a command
- when `Provide` may satisfy an `information_request`
- how to treat regulated content
- how to capture explicit decisions
- when to call `Get State`
- how to use broker-published vocabularies when forming structured payloads.

The transport may change how these instructions are delivered, but not their protocol meaning.

### 3.3 Expected MCP Client Bootstrap Flow

The expected bootstrap flow for an AI-capable MCP client is:

1. Discover tools, prompts, and resources from the MCP server.
2. Read or apply the broker instruction prompt.
3. Read the mandatory broker-published resources.
4. Begin invoking the canonical protocol tools.

### 3.4 Tool Naming and Namespace

The recommended MCP tool names are exactly the canonical protocol operation names in `snake_case`:

- `open_case`
- `provide`
- `get_state`
- `select`
- `resolve_action`
- `withdraw`.

The binding does not require a separate tool-name prefix. If a server exposes only this protocol surface, bare canonical names are preferred because they map cleanly to the transport-neutral specification. If a server exposes multiple unrelated tool families, a server-specific namespacing convention MAY be used, but it SHOULD preserve the recognisability of the canonical operation names.

### 3.5 Instruction and Resource Naming

The recommended naming pattern is:

- primary instruction prompt: `broker_instructions`
- mirrored instruction resource: `broker_instructions_resource`
- published vocabulary resources named by their canonical resource kinds.

The naming and addressing conventions across `§3.1` to `§3.5` define:

- the tool set corresponding to the canonical operation set
- how a client identifies and addresses cases across tool calls
- how mandatory broker-published resources are named, discovered, and refreshed
- how the primary broker instruction prompt is named and described
- how the mirrored instruction resource is named and refreshed.

## 4. Operation Mapping

This section maps each canonical protocol operation to an MCP tool.

### 4.1 Open Case

`Open Case` maps to:

- tool name: `open_case`
- input schema: `OpenCaseRequest`
- output schema: `CommandResponse`.

Semantic notes for MCP clients:

- this is the only tool that creates a new case identifier
- the returned `CommandResponse` is expected to contain that `case_id`
- the client SHOULD retain the returned `case_id` for all later case-scoped tools.

### 4.2 Provide

`Provide` maps to:

- tool name: `provide`
- input schema: `ProvideRequest`
- output schema: `CommandResponse`.

Semantic notes for MCP clients:

- use this tool for proactive structured data contribution
- use this tool to satisfy an active `information_request`
- do not use this tool to bypass any other pending broker action.

### 4.3 Get State

`Get State` maps to:

- tool name: `get_state`
- input schema: `GetStateRequest`
- output schema: `FullStateResponse`.

### 4.4 Select

`Select` maps to:

- tool name: `select`
- input schema: `SelectRequest`
- output schema: `CommandResponse`.

### 4.5 Resolve Action

`Resolve Action` maps to:

- tool name: `resolve_action`
- input schema: `ResolveActionRequest`
- output schema: `CommandResponse`.

### 4.6 Withdraw

`Withdraw` maps to:

- tool name: `withdraw`
- input schema: `WithdrawRequest`
- output schema: `CommandResponse`.

## 5. Schema and Description Model

This section defines how MCP schemas and descriptions carry the protocol.

Tool input and output schemas SHOULD align directly with the transport-neutral request and response schemas. At minimum:

- tool input schemas SHOULD mirror `OpenCaseRequest`, `ProvideRequest`, `GetStateRequest`, `SelectRequest`, `ResolveActionRequest`, and `WithdrawRequest`
- tool output schemas SHOULD mirror `CommandResponse` and `FullStateResponse`
- tool schemas SHOULD not rename protocol fields away from the shared canonical forms unless forced by MCP mechanics.

Tool descriptions have an important semantic role for AI-capable clients. They SHOULD communicate:

- the operation's purpose
- key preconditions
- gate-related restrictions
- when regulated content obligations matter
- when the client SHOULD prefer `Get State` over relying on local memory.

The broker instruction prompt and mirrored instruction resource are the primary source of broad behavioural guidance. Tool descriptions are the operation-local complement. They SHOULD reinforce the same rules rather than reinterpreting them differently.

Regulated-content handling obligations SHOULD be stated in both:

- the shared broker instruction content
- any tool descriptions where misuse is especially likely, such as `resolve_action`, `select`, or `provide`.

Pending broker actions and gates SHOULD be explained in tool descriptions using concrete language such as: the response may include `pending_action`; treat it as the active gate after the tool call.

## 6. MCP Error Mapping

This section defines how transport-neutral protocol errors are expressed through MCP tool failures.

For this release, the preferred MCP approach is:

- successful tool calls return the canonical success schema
- protocol failures are surfaced as MCP tool errors or failure structures that preserve the transport-neutral `error_code` semantics.

At minimum, the MCP binding MUST preserve the distinction between:

- malformed input
- unauthenticated access
- authenticated-but-unauthorised access
- blocked by pending action
- insufficient trust for direct User Agent handling
- invalid state transition
- invalid reference
- unknown or unsupported value
- stale or superseded broker action resolution
- content integrity failure
- internal processing failure.

An MCP error payload SHOULD preserve enough semantic information for the client to react correctly. At minimum, it SHOULD include, where applicable:

- `error_code`
- `message`
- `case_id`
- `action_id`
- `field`
- `reference_id`
- `required_scope`
- `required_trust_capability`
- `integrity_subject`.

The binding SHOULD avoid collapsing all failures into generic model-facing prose. A model-driven client must be able to tell whether it should:

- authenticate or refresh authority
- fix malformed input
- stop because a gate is active
- direct the user to the broker-controlled surface
- refresh state
- or treat the issue as broker-side failure.

## 7. Client Behaviour Expectations

This section defines the expectations of an MCP client or User Agent using the protocol through MCP.

The client SHOULD:

- apply the broker instruction prompt before using the tools
- read the broker-published resources before attempting to classify user input
- retain the returned `case_id` after `open_case`
- treat `pending_action` in a `CommandResponse` as the active post-command gate
- use `provide` to satisfy an active `information_request`
- use `resolve_action` only when the action expects an explicit formal resolution
- use `get_state` when local understanding may be stale, after reconnect, or when the user requests a full review
- treat regulated content as verbatim where the pending action indicates `regulated = true`
- keep access tokens, guest continuation credentials, and other secrets out of tool inputs, tool outputs, transcripts, and user-facing messages.

The client SHOULD interpret tool descriptions as operation-specific guidance under the broader broker instruction layer. If there appears to be tension between a narrow tool description and the broader broker instructions, the client SHOULD prefer the broader protocol rules and, where necessary, refresh state or seek clarification from the broker surface.

## 8. Authentication and Security

This section defines the MCP binding's initial realisation of the transport-neutral security model in [protocol-technical-spec.md](protocol-technical-spec.md#17-security-model).

### 8.1 Protected Tool and Resource Surface

Remote production MCP deployments MUST run over a secure transport appropriate to the deployment environment.

The MCP server MUST authenticate and authorise requests before allowing access to tools or resources that expose private case data, create or mutate case state, resolve broker actions, or retrieve case state. This includes `open_case` where the tool creates a private case that will later be continued by the same user authority.

Broker-published vocabulary resources, broker instructions, and tool descriptions MAY be available before account-bound or guest case authority is established. Public access to bootstrap resources MUST NOT imply access to case-scoped tools, private case resources, or regulated action resolution capability.

### 8.2 OAuth-Based Authorisation for Remote MCP

For production remote MCP deployments, the RECOMMENDED authorisation profile is the MCP authorisation profile using OAuth 2.1.

The broker MAY use JWT access tokens, opaque access tokens with server-side introspection, or another OAuth-compatible token representation. Regardless of token format, the MCP server MUST verify or resolve, at minimum:

- issuer or issuing authority
- intended audience or protected resource
- expiration
- scopes, permissions, or equivalent tool capability
- authenticated protocol participant identity (for example User Agent operator identity or platform client identity)
- account-bound user authority or guest case-continuation authority.

For account-bound cases, the token subject or resolved token state identifies a broker-recognised user account or equivalent durable user authority.

For guest cases, the token subject or resolved token state identifies a pseudonymous guest participant, guest session, or guest case-continuation authority. A guest OAuth token MUST NOT be treated as proof that the broker knows the user's durable real-world identity.

The broker MAY bind case access either by carrying case-specific authorisation in the token or by resolving the token subject to a server-side access-control record. In either model, `case_id` remains an identifier only and MUST NOT authorise access by itself.

Access tokens, refresh tokens, guest continuation credentials, and other secrets MUST NOT be carried in MCP tool inputs, MCP tool outputs, transcripts, broker-authored content, model-visible resource content, or user-facing prose. They belong in the MCP transport's authentication channel or another binding-defined credential channel.

Tool descriptions and metadata SHOULD declare the authentication schemes and scopes required for each protected tool. Tool-specific scopes or permissions SHOULD be narrow enough to distinguish reading case state, providing data, making selections, resolving broker actions, withdrawing, and reading public resources.

When authentication is missing, expired, or insufficient, the MCP server SHOULD use the binding's native authentication challenge mechanism so the client can authenticate or refresh authority. The MCP error payload SHOULD still preserve the transport-neutral `unauthenticated` or `unauthorised` error meaning.

### 8.3 Hosted MCP Clients and ChatGPT Apps

Where the MCP binding is deployed through a hosted MCP client such as ChatGPT Apps, the broker SHOULD distinguish three identities:

- the hosted MCP client or platform
- the end-user authority represented by the OAuth token
- the User Agent or app capability profile the broker uses for trust calibration.

The broker SHOULD authenticate or identify the hosted MCP client or platform using mechanisms supported by that platform, such as platform-managed client identity, mutual TLS, signed client metadata, or another documented platform control. The broker SHOULD NOT rely only on a per-session dynamic OAuth `client_id` as the stable identity for User Agent certification, benchmarking, or long-term trust calibration.

The OAuth user authority may still be account-bound or guest-bound. A ChatGPT App or similar hosted MCP client can therefore call the protocol with a guest OAuth token whose subject represents a pseudonymous guest participant rather than a broker-recognised account holder.

### 8.4 Case Access Control and Guest Continuation

The broker MUST apply the same case access-control decision across all tools and resources that operate on the same case. A caller that cannot access `get_state` for a case MUST NOT be able to infer or mutate the same case through `provide`, `select`, `resolve_action`, or `withdraw`.

Guest continuation credentials SHOULD be short-lived, case-scoped, revocable, and bound to the authenticated MCP client or session context where practical. A broker SHOULD require re-authentication, token refresh, account binding, identity verification, or broker-controlled surface completion before allowing a guest case to perform higher-risk actions.

### 8.5 Trust-Calibrated Capability

MCP authentication and OAuth scopes establish authority to call tools. They do not by themselves establish that the User Agent is trusted to handle every regulated moment directly.

The broker MUST still apply trust-calibrated capability policy when processing `select`, `resolve_action`, and any operation that could cause or complete a regulated pending broker action. If the authenticated participant lacks the required trust capability, the broker SHOULD direct the user to the broker-controlled surface where possible. If the participant attempts an operation it is not trusted to perform directly, the broker SHOULD return `trust_level_insufficient`.

### 8.6 Regulated Content Integrity

For this release, MCP protects broker-authored regulated content in transit through the secure channel used by the deployment and preserves auditability through stable action identifiers and broker evidence records. A broker SHOULD record the exact regulated content issued for each pending broker action, together with its action identifier, timestamp, and resolution outcome.

A broker MAY add stronger regulated-content integrity controls, such as content digests, detached signatures, signed tool-result envelopes, or attestation mechanisms. If such controls are used, they SHOULD be documented in the relevant tool descriptions, broker instructions, or resource metadata, and failures SHOULD map to `content_integrity_failure`.

The MCP binding does not yet mandate one regulated-content signing or attestation mechanism. That remains a production-profile question for later versions.

## 9. Worked MCP Examples

This section mirrors the same case flow used in the REST binding:

- `open_case` creates a debt-consolidation case
- the broker returns a blocking `information_request`
- `provide` supplies the missing facts and produces a plan
- `select` chooses the plan and triggers a regulated `disclosure`
- `resolve_action` acknowledges the disclosure
- `get_state` retrieves the full current case projection.

The examples are illustrative. The important point is that the tool inputs and outputs align directly with the transport-neutral schemas and preserve the same gate behaviour as the REST flow. Authentication and authorisation material is omitted from the examples.

### 9.1 `open_case`

Tool input:

```json
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

Tool result:

```json
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

### 9.2 `provide`

Tool input:

```json
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

Tool result:

```json
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

### 9.3 `select` Triggering a Regulated Gate

Tool input:

```json
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

Tool result:

```json
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

### 9.4 `resolve_action`

Tool input:

```json
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

Tool result:

```json
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

### 9.5 `get_state`

Tool input:

```json
{
  "case_id": "case_001"
}
```

Tool result:

```json
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

## 10. Open MCP Questions

The remaining MCP-specific questions for later refinement are:

- how strongly the binding should rely on schema versus narrative tool descriptions for AI-capable clients
- whether any MCP-specific affordance should change how pending actions are surfaced beyond the current `pending_action` model
- how MCP resource versioning and cache invalidation should work in later versions
- whether later versions still need a mirrored instruction resource once prompt-delivery patterns are better established.
