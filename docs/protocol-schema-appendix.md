# Agentic Credit Broking Protocol Schema Appendix

## Status

This document defines the shared transport-neutral schema catalogue for the Agentic Credit Broking Protocol.

Its purpose is to identify the protocol structures that REST and MCP should
share in common, and to describe the boundary between canonical protocol
schemas and binding-specific transport wrappers.

This appendix is a companion to:

- [protocol-standards-and-conventions.md](protocol-standards-and-conventions.md)
- [protocol-technical-spec.md](protocol-technical-spec.md)
- [protocol-rest-binding.md](protocol-rest-binding.md)
- [protocol-mcp-binding.md](protocol-mcp-binding.md)

## 1. Purpose

This appendix defines one transport-neutral schema set for the semantic objects of the protocol. Both the REST and MCP bindings map to that common set.

The point of this appendix is to prevent a hidden split where the REST binding
and the MCP binding describe slightly different protocol objects. If two
bindings expose the same protocol semantics, they should share the same core
schema definitions wherever the field set affects protocol meaning.

## 2. Schema Boundary Rule

The boundary rule for this document set is:

- if a field changes the meaning of the protocol, it belongs in the shared
  canonical schema set
- if a field exists only because of HTTP or MCP mechanics, it belongs in the
  relevant binding.

This means the shared schema set defines the shape of protocol meaning, while the binding documents define the transport mechanics that carry those shapes.

## 3. Canonical Schema Set

This release defines a common schema set for at least the following transport-neutral structures:

- `OpenCaseRequest`
- `ProvideRequest`
- `GetStateRequest`, if `Get State` is given an explicit request shape beyond a
  case identifier
- `SelectRequest`
- `ResolveActionRequest`
- `WithdrawRequest`
- `CommandResponse`
- `FullStateResponse`
- `Event`
- `PendingBrokerAction`
- `BrokerActionResolution`
- `EntityReference`
- `PublishedResource`
- the mandatory broker-published resource payloads and entry shapes
- the initial core entity schemas needed for interoperability.

## 4. Binding-Specific Structures

The following remain outside the shared canonical schema set unless a later revision decides otherwise.

### 4.1 REST-Specific Structures

These include:

- endpoint paths
- HTTP methods
- request headers
- response headers
- HTTP status code mappings
- any REST-only response envelope introduced for HTTP convenience rather than
  protocol meaning.

### 4.2 MCP-Specific Structures

These include:

- tool names
- tool descriptions
- any MCP-specific server namespace choices
- MCP-specific tool wrapper metadata that exists for MCP operation rather than
  protocol semantics.

## 5. Schema Publication Model

This document set separates human-readable schema explanation from machine-readable schema artefacts.

### 5.1 Human-Readable Schema Definition

This appendix describes:

- the canonical schema names
- the purpose of each schema
- the main fields each schema contains
- the relationships between schemas
- which areas remain provisional in this release.

### 5.2 Machine-Readable Schema Artefacts

If machine-readable schemas are published as part of this release, they should live in a transport-neutral location alongside this appendix rather than inside either binding document.

A plausible workspace layout would be:

- `docs/protocol-schema-appendix.md`
- `docs/protocol-schemas/`

The exact schema language to publish normatively remains an open question.
This release may describe the structures first and introduce formal JSON Schema files
as a companion artefact once the field models have stabilised.

## 6. Initial Schema Catalogue

This section is the working catalogue of the shared canonical schemas.

### 6.1 Operation Request Schemas

This section defines the canonical request schema family for the six protocol operations.

#### 6.1.1 Shared Request Principles

This release uses per-operation request schemas rather than a universal request envelope.

Across operation request schemas:

- mutating requests other than `Open Case` SHOULD contain `case_id`
- references to broker-defined vocabularies SHOULD use identifiers from published resources
- opaque transcript fields MAY be present where the operation semantics allow them
- decision-bearing semantics MUST be carried through structured fields rather than free text.

#### 6.1.2 `OpenCaseRequest`

`OpenCaseRequest` SHOULD allow case creation with or without initial contribution material.

At minimum, it MAY contain:

- `initial_facts`
- `initial_party_attributes`
- `initial_goals`
- `initial_evidence`
- `transcript`
- `opening_context`.

Field expectations:

- `initial_facts`: array of `FinancialFact` input objects
- `initial_party_attributes`: array of `PartyAttribute` input objects
- `initial_goals`: array of `FinancialGoal` input objects
- `initial_evidence`: array of `Evidence` input objects or references
- `transcript`: optional opaque audit text
- `opening_context`: optional broker-specific extension object.

#### 6.1.3 `ProvideRequest`

`ProvideRequest` targets an existing case and carries one or more contribution items.

At minimum, it SHOULD contain:

- `case_id`
- at least one of `facts`, `party_attributes`, `goals`, or `evidence`
- `transcript`.

Field expectations:

- `facts`: array of `FinancialFact` input objects
- `party_attributes`: array of `PartyAttribute` input objects
- `goals`: array of `FinancialGoal` input objects
- `evidence`: array of `Evidence` input objects or references
- `transcript`: optional opaque audit text.

#### 6.1.4 `GetStateRequest`

`GetStateRequest` is intentionally minimal in this release.

It SHOULD contain:

- `case_id`.

Bindings MAY carry case identity outside the body or tool-input object where natural, but the transport-neutral request concept is simply the request to retrieve one case by case identifier.

#### 6.1.5 `SelectRequest`

`SelectRequest` targets an existing case and names one selected entity.

It SHOULD contain:

- `case_id`
- `selection`.

`selection` SHOULD be an object containing:

- `entity_ref`
- optional `transcript`.

#### 6.1.6 `ResolveActionRequest`

`ResolveActionRequest` targets an existing pending broker action and supplies its formal resolution.

It SHOULD contain:

- `case_id`
- `action_id`
- `resolution`.

`resolution` SHOULD conform to `BrokerActionResolution`.

#### 6.1.7 `WithdrawRequest`

`WithdrawRequest` targets an existing case and communicates withdrawal.

It SHOULD contain:

- `case_id`
- optional `transcript`
- optional `reason_summary`.

### 6.2 Response Schemas

This section defines the shared response schemas used by the transport-neutral protocol.

#### 6.2.1 `CommandResponse`

`CommandResponse` is the canonical success response schema for all mutating operations.

At minimum, `CommandResponse` SHOULD contain:

- `case_id`
- `events`
- `pending_action`
- `request_id`
- `sequence`
- `generated_at`.

Field expectations:

- `case_id`: identifier of the case to which the command result applies
- `events`: ordered array of `Event`
- `pending_action`: `PendingBrokerAction` or `null`
- `request_id`: optional correlation identifier
- `sequence`: optional case-local sequence marker after command processing
- `generated_at`: optional RFC `3339` timestamp.

The schema SHOULD allow `events` to be empty. The schema SHOULD treat `pending_action` as the post-command active gate rather than merely a newly issued action.

#### 6.2.2 `FullStateResponse`

`FullStateResponse` is the canonical response schema for `Get State`.

At minimum, `FullStateResponse` SHOULD contain:

- `case_id`
- `case_status`
- `entities`
- `pending_action`
- `last_sequence`
- `generated_at`
- `resource_versions`.

Field expectations:

- `case_id`: identifier of the case being projected
- `case_status`: current lifecycle status
- `entities`: current protocol-visible entity set or projection
- `pending_action`: `PendingBrokerAction` or `null`
- `last_sequence`: optional latest visible case sequence marker
- `generated_at`: RFC `3339` timestamp for the projection
- `resource_versions`: optional object keyed by published resource kind to version string.

If `resource_versions` is present, it MAY include a subset of published resource kinds. Omitted kinds mean no version marker is supplied for that kind in that response.

#### 6.2.3 Shared Response Metadata

The schema appendix SHOULD treat the following as shared response-level metadata concepts:

- `case_id`
- sequencing markers such as `sequence` or `last_sequence`
- `generated_at`
- optional correlation metadata such as `request_id`.

These shared fields MAY be factored into reusable schema fragments if machine-readable schemas are later published.

### 6.3 Event Schemas

This section defines the common `Event` schema and the initial event catalogue.

#### 6.3.1 `Event`

At minimum, `Event` SHOULD contain:

- `id`
- `event_type`
- `case_id`
- `occurred_at`
- `sequence`
- `entity_refs`
- `entities`.

Field expectations:

- `id`: stable event identifier
- `event_type`: one of the defined protocol event types
- `case_id`: owning case identifier
- `occurred_at`: RFC `3339` timestamp
- `sequence`: case-local ordering marker
- `entity_refs`: array of `EntityReference`
- `entities`: array of embedded typed entities or entity payload fragments.

The schema SHOULD require that `entity_refs` and `entities` be arrays even when empty. The schema MAY allow `entities` to contain only the subset of embedded entities needed for meaningful interpretation of the event.

#### 6.3.2 Event Type Enum

The initial `event_type` enum SHOULD include:

- `profile_updated`
- `goals_updated`
- `plans_ready`
- `plans_revised`
- `offers_ready`
- `offers_revised`
- `case_status_changed`.

#### 6.3.3 Event-Type Payload Guidance

At minimum, event-specific payload fragments SHOULD remain light. The common event envelope plus embedded entities is preferred over a large family of event-specific top-level payload objects.

Where an implementation needs event-specific detail, that detail SHOULD normally be carried through:

- embedded entities in `entities`
- optional summary fields whose meaning is stable across bindings.

If later revisions introduce event-specific payload fragments, they SHOULD layer on top of the common `Event` schema rather than replacing it.

### 6.4 Pending Broker Action Schemas

This section defines the canonical schema family for currently active broker actions.

#### 6.4.1 `PendingBrokerAction`

At minimum, `PendingBrokerAction` SHOULD contain:

- `id`
- `action_type`
- `case_id`
- `issued_at`
- `blocking`
- `regulated`
- `content`
- `content_format`
- `response_expectation`
- `subjects`.

Field expectations:

- `id`: stable action identifier
- `action_type`: one of the defined protocol action types
- `case_id`: owning case identifier
- `issued_at`: RFC `3339` timestamp
- `blocking`: boolean
- `regulated`: boolean
- `content`: broker-authored content string or `null`
- `content_format`: optional content-format indicator
- `response_expectation`: one of the defined protocol response expectations
- `subjects`: array of `ActionSubject`.

`PendingBrokerAction` MAY additionally contain action-type-specific fields such as:

- `information_scope`
- `purpose`
- `consent_required`
- `consequences`
- `destination_url`
- `destination_name`
- `target_case_status`.

#### 6.4.2 `ActionSubject`

`ActionSubject` is the schema used in the `subjects` array of a pending broker action.

At minimum, `ActionSubject` SHOULD contain:

- `entity_ref`
- `subject_role`.

It MAY also contain:

- `label`
- `summary`.

Field expectations:

- `entity_ref`: `EntityReference`
- `subject_role`: one of the protocol-fixed subject roles
- `label`: optional short human-readable label
- `summary`: optional short human-readable summary.

#### 6.4.3 Subject Role Enum

The `subject_role` enum SHOULD include:

- `primary`
- `selected_option`
- `related_goal`
- `related_plan`
- `related_offer`
- `supporting_entity`.

#### 6.4.4 Response Expectation Enum

The `response_expectation` enum SHOULD include:

- `provide_information`
- `acknowledge`
- `grant_or_refuse`
- `affirm_or_deny`
- `authorise_or_refuse`
- `confirm_direction`
- `none`.

#### 6.4.5 Action Type Enum

The `action_type` enum SHOULD include:

- `information_request`
- `disclosure`
- `consent`
- `declaration`
- `instruction`
- `case_outcome`.

#### 6.4.6 `InformationScope`

When `action_type = information_request`, the action MAY contain `information_scope`.

At minimum, `InformationScope` SHOULD be an object containing:

- `needs`: ordered array of `InformationNeed`.

Each `InformationNeed` SHOULD contain:

- `item_type`
- `information_context`
- `target_entity_ref`.

Field expectations:

- `item_type`: broker-recognised information item identifier
- `information_context`: one of the protocol-fixed information contexts
- `target_entity_ref`: optional `EntityReference`, typically used for goal-related requests.

#### 6.4.7 Action-Type-Specific Fragments

The following lightweight action-type-specific fragments are sufficient:

- `ConsentActionFields`: `purpose`, `consent_required`
- `InstructionActionFields`: `consequences`, `destination_url`, `destination_name`
- `CaseOutcomeActionFields`: `target_case_status`.

The appendix SHOULD avoid introducing a completely separate top-level schema per action type unless that becomes necessary in a later revision.

#### 6.4.8 `BrokerActionResolution`

`BrokerActionResolution` is the canonical schema for a formal response to a pending broker action.

At minimum, it SHOULD contain:

- `outcome`
- `responded_at`
- `transcript`.

It MAY also contain:

- `notes`
- `provided_contribution_id`.

Field expectations:

- `outcome`: one of the allowed response outcomes for the targeted action
- `responded_at`: REQUIRED RFC `3339` timestamp
- `transcript`: optional opaque audit text
- `notes`: optional short non-decision-bearing summary
- `provided_contribution_id`: optional identifier linking an information-request resolution to the contribution that supplied the requested information.

The allowed `outcome` enum SHOULD include:

- `acknowledged`
- `granted`
- `refused`
- `affirmed`
- `denied`
- `authorised`
- `direction_confirmed`.

### 6.5 Entity Reference and Entity Schemas

This section defines the common reference shape and the initial concrete entity schema family for interoperability.

#### 6.5.1 `EntityReference`

At minimum, `EntityReference` SHOULD contain:

- `id`
- `entity_type`.

It MAY also contain:

- `label`
- `summary`.

Field expectations:

- `id`: stable entity identifier
- `entity_type`: one of the recognised protocol entity types
- `label`: optional short display label
- `summary`: optional short display summary.

#### 6.5.2 Common Embedded Entity Shape

Every embedded entity SHOULD contain:

- `id`
- `entity_type`
- type-specific fields.

The appendix refers to this pattern as the common embedded entity shape. It is a modelling rule, not necessarily a separate wire object named `Entity`.

#### 6.5.3 Initial Entity Type Enum

The initial `entity_type` enum SHOULD include at least:

- `financial_profile`
- `financial_fact`
- `party_attribute`
- `financial_goal`
- `evidence`
- `financial_plan`
- `financial_plan_suitability_assessment`
- `financial_plan_recommendation`
- `product_offer`
- `product_offer_suitability_assessment`
- `product_offer_recommendation`
- `case_contribution`
- `case_interaction_response`.

#### 6.5.4 Initial Concrete Entity Schemas

The appendix SHOULD define lightweight initial entity schemas for the minimum set above.

At a minimum, the following field expectations apply:

- `FinancialProfile`: `id`, `entity_type`, `facts`
- `FinancialFact`: `id`, `entity_type`, `fact_type_id`, `value`, `operator`, `information_context`
- `PartyAttribute`: `id`, `entity_type`, `attribute_type_id`, `value`
- `FinancialGoal`: required `id`, `entity_type`, `goal_type_id`, `goal_statement`; optional `target_timeframe`, `desired_change_facts`
- `Evidence`: `id`, `entity_type`, `evidence_type_id`
- `FinancialPlan`: required `id`, `entity_type`, `steps`; optional linked assessments and recommendations by reference or embedding
- `FinancialPlanSuitabilityAssessment`: `id`, `entity_type`, `assessment_summary`, optional `assessment_rationale`
- `FinancialPlanRecommendation`: `id`, `entity_type`, `recommendation_summary`, optional `recommendation_rationale`
- `ProductOffer`: `id`, `entity_type`, `product_category_id`, `feature_values`
- `ProductOfferSuitabilityAssessment`: `id`, `entity_type`, `assessment_summary`, optional `assessment_rationale`
- `ProductOfferRecommendation`: `id`, `entity_type`, `recommendation_summary`, optional `recommendation_rationale`
- `CaseContribution`: `id`, `entity_type`, optional `transcript`, contribution member references or embeddings
- `CaseInteractionResponse`: `id`, `entity_type`, `outcome`, optional `transcript`.

These are minimum interoperability fields, not exhaustive domain models.

`FinancialPlan.steps` SHOULD be an array of strings intended for user-facing presentation. In this release, these strings are display-oriented guidance and MUST NOT be interpreted by the User Agent as standalone decision-bearing protocol semantics.

For numeric serialisation, financial decimal values (for example money amounts, percentages, and rates) SHOULD be serialised as strings to avoid precision loss across transports. Pure count values (for example term months) MAY be serialised as JSON numbers.

### 6.6 Error Payload Schemas

This section defines the transport-neutral structured error payload model that bindings must preserve semantically even when they express errors through different transport mechanisms.

#### 6.6.1 `ProtocolError`

At minimum, a transport-neutral error payload SHOULD contain:

- `error_code`
- `message`.

It MAY also contain:

- `case_id`
- `action_id`
- `field`
- `reference_type`
- `reference_id`
- `required_scope`
- `required_trust_capability`
- `integrity_subject`
- `can_retry`.

Field expectations:

- `error_code`: one of the protocol-defined error categories
- `message`: short human-readable explanation
- `case_id`: relevant case identifier when applicable
- `action_id`: relevant pending action identifier when applicable
- `field`: request field implicated in the failure, when useful
- `reference_type`: kind of bad or missing reference
- `reference_id`: identifier of the bad or missing reference
- `required_scope`: optional binding-level scope or permission needed for the operation, when safe to disclose
- `required_trust_capability`: optional operation or action capability the broker requires before direct User Agent handling, when safe to disclose
- `integrity_subject`: optional identifier for the content, field, action, or payload whose integrity check failed, when safe to disclose
- `can_retry`: optional boolean hint indicating whether the same request may be attempted again.

#### 6.6.2 Error Code Enum

The `error_code` enum SHOULD include:

- `malformed_request`
- `unauthenticated`
- `unauthorised`
- `unauthorized` (compatibility alias of `unauthorised`)
- `invalid_reference`
- `invalid_operation_for_state`
- `blocked_by_pending_action`
- `trust_level_insufficient`
- `unknown_or_unsupported_value`
- `content_integrity_failure`
- `stale_or_superseded_action`
- `concurrency_or_sequence_conflict`
- `internal_processing_failure`.

#### 6.6.3 Error-Payload Guidance

Bindings MAY enrich their native error structures, but the payload semantics above SHOULD remain recoverable across bindings.

In particular:

- `blocked_by_pending_action` SHOULD include `action_id` when an active gate is known
- `unauthenticated` SHOULD include enough binding-native challenge information to let the client authenticate, but that challenge information may live outside `ProtocolError`
- `unauthorised` SHOULD avoid disclosing protected case or entity existence to callers that lack access
- `unauthorized` SHOULD be treated as a compatibility alias of `unauthorised` where ecosystems or middleware expect American spelling
- `trust_level_insufficient` SHOULD identify the missing operation or action capability where that can be done safely
- `content_integrity_failure` SHOULD identify the affected content or payload where that can be done safely
- `stale_or_superseded_action` SHOULD include `action_id` when that does not disclose protected case information
- `invalid_reference` SHOULD identify the bad reference where that can be done safely
- `unknown_or_unsupported_value` SHOULD identify the offending field or identifier when practical.

### 6.7 Published Resource Schemas

This section defines the common schema approach for broker-published resources in this release.

#### 6.7.1 Discovery Boundary

This release does not define a transport-neutral resource catalogue or discovery document. Resource discovery remains binding-specific in this release.

The transport-neutral schema set therefore defines the payload shapes of published resources and their entries, but not a universal discovery envelope. The REST binding may expose discovery through HTTP endpoint structure or linked documentation. The MCP binding may expose discovery through MCP resource listing or other MCP-native mechanisms.

#### 6.7.2 Common `PublishedResource` Shape

Every mandatory broker-published resource SHOULD conform to a common high-level shape:

- `resource_id`: stable identifier for the published resource within the broker surface
- `resource_kind`: the kind of catalogue being published
- `title`: human-readable title for the resource
- `description`: rich description of the resource as a whole
- `version`: broker-assigned version string for cache or change tracking
- `generated_at`: RFC `3339` timestamp indicating when this payload version was generated
- `entries`: array of resource entries of the kind implied by `resource_kind`.

In prose, this common shape is referred to as `PublishedResource`.

`resource_kind` for the mandatory set SHOULD use one of:

- `financial_fact_types`
- `party_attribute_types`
- `goal_types`
- `financial_product_features`
- `financial_product_categories`
- `evidence_types`.

#### 6.7.3 Common Entry Fragments

Every entry in a mandatory broker-published resource SHOULD share a common semantic fragment:

- `id`: stable broker-defined identifier for the catalogue entry
- `label`: concise human-readable label
- `description`: rich semantic description suitable for AI-capable clients and human readers
- `examples`: array of natural-language examples showing how a user may express the concept
- `status`: optional lifecycle marker for the entry, such as `active` or `deprecated`.

`description` SHOULD explain inclusion and exclusion boundaries where confusion with nearby concepts is plausible.

`examples` SHOULD be short, realistic user utterances rather than abstract labels.

#### 6.7.4 `FinancialFactTypesResource`

The resource published under `resource_kind = financial_fact_types` SHOULD contain entries of type `FinancialFactTypeEntry`.

Each `FinancialFactTypeEntry` SHOULD include:

- the common entry fragment
- `value_type`: the `FinancialFactValueType` expected for facts of this type
- `supported_operators`: array of protocol-fixed value operators that may be used with this fact type
- `permitted_information_contexts`: array of protocol-fixed information contexts in which this fact type may be provided
- `period_kind`: optional indicator for periodic facts such as `weekly`, `monthly`, `annual`, or `other`
- `currency_required`: optional boolean indicating whether money values of this type normally require a currency code.

Example `value_type` values include:

- `money`
- `integer`
- `enumeration`
- `percentage`
- `boolean`
- `duration`
- `date`
- `text`.

#### 6.7.5 `PartyAttributeTypesResource`

The resource published under `resource_kind = party_attribute_types` SHOULD contain entries of type `PartyAttributeTypeEntry`.

Each `PartyAttributeTypeEntry` SHOULD include:

- the common entry fragment
- `value_type`: the `PartyAttributeValueType` expected for this attribute type
- `applicable_party_kinds`: optional array indicating whether the attribute applies to `person`, `organisation`, or both
- `supports_time_bounds`: optional boolean indicating whether repeated values of this type commonly form a time-bounded history
- `sensitivity`: optional marker such as `normal`, `sensitive`, or `regulated_identity`.

Example `value_type` values include:

- `text_value`
- `date_value`
- `integer_value`
- `boolean_value`
- `enumeration_value`
- `person_name_value`
- `organisation_name_value`
- `postal_address_value`
- `telecommunication_number_value`
- `identifier_value`.

#### 6.7.6 `GoalTypesResource`

The resource published under `resource_kind = goal_types` SHOULD contain entries of type `GoalTypeEntry`.

Each `GoalTypeEntry` SHOULD include:

- the common entry fragment
- `typical_constraint_fact_types`: optional array of `financial_fact_types` entry identifiers commonly used to express desired changes for this goal type
- `supports_target_timeframe`: optional boolean indicating whether a timeframe is commonly relevant
- `goal_statement_guidance`: optional short guidance for how a User Agent should frame an opaque `goal_statement` when the user has expressed this kind of goal.

This resource exists to help the User Agent decide when a user's stated intent is best represented as one goal type rather than another, and how the structured desired-change constraints commonly attach to that goal.

#### 6.7.7 `FinancialProductFeaturesResource`

The resource published under `resource_kind = financial_product_features` SHOULD contain entries of type `FinancialProductFeatureEntry`.

Each `FinancialProductFeatureEntry` SHOULD include:

- the common entry fragment
- `feature_value_type`: the `FeatureValueType` expected for this product feature
- `comparable`: optional boolean indicating whether values of this feature are generally meaningful for cross-offer comparison
- `presentation_hints`: optional array of short hints relevant to user-facing presentation, such as `percentage`, `money`, `term_length`, or `warning_sensitive`.

Example `feature_value_type` values include:

- `percentage`
- `money`
- `duration`
- `integer`
- `boolean`
- `enumeration`.

#### 6.7.8 `FinancialProductCategoriesResource`

The resource published under `resource_kind = financial_product_categories` SHOULD contain entries of type `FinancialProductCategoryEntry`.

Each `FinancialProductCategoryEntry` SHOULD include:

- the common entry fragment
- `parent_category_id`: optional identifier of the parent category when the category is part of a hierarchy
- `path`: optional ordered array of ancestor and self identifiers to simplify client-side navigation
- `leaf`: optional boolean indicating whether the category currently has children in the published catalogue.

This resource provides the broker's product-family taxonomy for planning, sourcing, and category-based requirement expression.

#### 6.7.9 `EvidenceTypesResource`

The resource published under `resource_kind = evidence_types` SHOULD contain entries of type `EvidenceTypeEntry`.

Each `EvidenceTypeEntry` SHOULD include:

- the common entry fragment
- `supports_subject_kinds`: optional array indicating whether the evidence type commonly substantiates `financial_fact`, `party_attribute`, or both
- `document_like`: optional boolean indicating whether the evidence is primarily document-based as opposed to feed-based or declaration-based
- `coverage_period_supported`: optional boolean indicating whether this evidence type commonly carries a relevant coverage period.

This resource helps the User Agent classify evidence consistently when the user uploads, references, or describes supporting material.

#### 6.7.10 Shared Field Semantics

The following field semantics apply across the mandatory published-resource schemas:

- identifiers in resource entries are broker-defined and stable within the broker's published surface
- resource entry identifiers MAY be referenced by operation payloads where the canonical protocol schema chooses identifier-based vocabulary linkage
- all examples are illustrative and not decision-bearing
- natural-language descriptions and examples are intended to help an AI-capable User Agent classify user input, but they do not override the structural constraints of the protocol schemas.

#### 6.7.11 Example Minimal Resource Skeleton

The following illustrative shape shows the minimum pattern expected of a published resource in this release:

```json
{
  "resource_id": "financial_fact_types",
  "resource_kind": "financial_fact_types",
  "title": "Financial Fact Types",
  "description": "The financial fact types recognised by this broker.",
  "version": "2026-04-23",
  "generated_at": "2026-04-23T12:00:00Z",
  "entries": [
    {
      "id": "income_net_monthly",
      "label": "Income.NetMonthly",
      "description": "Net monthly income received by the applicant after deductions.",
      "examples": ["I take home about 2,800 a month."],
      "value_type": "money",
      "supported_operators": ["equal_to", "approximately", "between"],
      "permitted_information_contexts": ["current_profile", "goal_constraint"]
    }
  ]
}
```

## 7. Open Questions

The remaining schema-specific questions are:

- which schema language is normative in this release
- how strict validation should be
- whether request envelopes are needed or whether per-operation shapes are
  sufficient
- how much of the conceptual model should be represented in the first schema
  set
- how embedded entities and references should be balanced in response payloads
- whether resource-entry cross references should remain identifier-based only or support richer typed links
- how much validation should be placed on broker-authored descriptions and examples versus treating them as advisory text.

## 8. Relationship to the Binding Docs

The REST and MCP binding documents reference this appendix when they map protocol operations to transport-specific interfaces.

They should not redefine the shared semantic fields of canonical protocol objects unless a later revision explicitly changes the transport-neutral specification.
