# Agentic Credit Broking Protocol Technical Specification

## Status

This document is the transport-neutral technical specification for the Agentic Credit Broking Protocol.

It defines the protocol semantics independently of any specific interface technology. Transport-specific realisations are defined separately in:

- [protocol-rest-binding.md](protocol-rest-binding.md)
- [protocol-mcp-binding.md](protocol-mcp-binding.md)

This specification is intentionally provisional. It is concrete enough to support prototype implementation and technical review, but it is not finalised for production implementation, stable compatibility guarantees, or full conformance assessment.

## 1. Purpose

The purpose of this document is to define the first implementable technical form of the protocol described conceptually in [whitepaper.md](whitepaper.md). It provides a shared semantic contract that can be realised through multiple interface styles without changing the underlying protocol model.

This specification is intended to make the protocol tangible, testable, and open to engineering scrutiny while preserving room for revision in later versions.

## 2. Scope

This specification defines:

- the canonical operation set
- the shared request and response semantics
- the event model
- the pending broker action and gate model
- the broker vocabulary and published resource model
- the initial rules for identifiers and references
- the initial lifecycle and progression rules
- the initial error model
- the initial transport-neutral security model
- the semantic contract that transport bindings must preserve.

This specification does not yet define complete production security profiles, final conformance rules, or complete domain vocabularies for every use case or jurisdiction.

## 3. Design Principles

This technical specification preserves the core interaction shape introduced in the whitepaper:

- The User Agent drives the interaction by choosing when to invoke protocol operations.
- The broker controls progression through blocking broker actions, referred to in this specification as gates.
- Structured protocol entities and opaque user-agent-side material remain distinct.
- Regulated broker-controlled content must remain identifiable and preservable across bindings.

Additional technical principles are:

- Transport neutrality: the protocol semantics MUST NOT depend on REST- or MCP-specific behaviour.
- Binding equivalence: the REST and MCP bindings must expose materially equivalent semantics.
- Prototype implementability: payloads and interaction rules SHOULD be specific enough for independent prototype implementations.
- Intentional incompleteness: unresolved areas SHOULD be marked explicitly rather than hidden behind vague prose.

## 4. Conformance Status of This Release

This is a draft release. Implementations may use it for prototyping, simulation, interoperability experiments, and specification refinement.

Implementers must not assume:

- backward compatibility across subsequent revisions
- complete domain coverage
- final transport design
- final binding-specific security profiles or authentication mechanisms
- final certification or conformance obligations.

## 5. Protocol Model Overview

The protocol is a request-response contract in which the User Agent chooses which protocol operation to invoke and the broker reports the resulting protocol-visible state changes. The broker does not drive the conversation by issuing unsolicited next steps. Instead, it processes User Agent commands, emits protocol events describing what changed, and may raise a pending broker action when further progression is blocked until the user responds.

The `Case` is the primary lifecycle container of the protocol. All protocol-visible contributions, selections, pending broker actions, broker action resolutions, events, and lifecycle transitions occur within exactly one case. A case therefore provides the stable boundary within which the broker accumulates evidence, tracks current state, and determines whether progression is possible.

The protocol defines two interaction shapes. `Open Case`, `Provide`, `Select`, `Resolve Action`, and `Withdraw` are command-like operations. They ask the broker to process an instruction or contribution and return a `CommandResponse`. `Get State` is a retrieval operation. It does not mutate protocol state, does not emit protocol events, and returns a `FullStateResponse` representing the current broker projection of the case.

Events and full state serve different purposes. Events are delta-oriented summaries of protocol-visible changes caused by a command. They are intended to give the User Agent everything needed to present what changed without requiring an immediate follow-up read. Full state is a projection-oriented representation of the current case, used when the User Agent needs the complete current picture rather than just the most recent changes.

A pending broker action is the protocol mechanism by which the broker controls gates. When present and unresolved, it indicates that the broker requires a defined user-side response before certain forms of case progression may continue. A gate blocks command progression except where the protocol explicitly allows `Provide` to satisfy an `information_request` by supplying the requested structured data.

The transport-neutral model is the semantic source of truth. REST and MCP bindings may package the protocol differently, but they MUST preserve the same operation meanings, state transitions, event meanings, pending-action semantics, and error semantics. A binding is conformant to this specification only to the extent that it realises those shared semantics rather than inventing transport-specific protocol behaviour.

## 6. Core Concepts

This section defines the transport-neutral meaning of the protocol's core technical concepts. These definitions align with the concepts introduced in the whitepaper but are stated here in implementation-oriented terms for binding, schema, and interoperability work.

### 6.1 Case

A `Case` is the broker-managed engagement unit for one protocol journey. It is the scope within which the broker receives structured contributions, develops plans, sources offers, raises broker actions, records user decisions, and ultimately reaches a terminal outcome or withdrawal. A case has a stable identifier, a lifecycle status, a current state projection, and an evidence record that accumulates protocol-visible history.

### 6.2 Operation

An `Operation` is a named protocol action that the User Agent invokes against the broker. Each operation has defined input semantics, preconditions, processing expectations, and response semantics. The canonical operation set for this release is `open_case`, `provide`, `get_state`, `select`, `resolve_action`, and `withdraw`.

### 6.3 Command Response

A `CommandResponse` is the shared transport-neutral response returned by every mutating operation. It reports the protocol-visible result of processing a command rather than the broker's full internal work log. At minimum, it contains the events emitted by the command, any newly active pending broker action, and correlation or sequencing metadata needed to interpret the response safely.

### 6.4 Event

An `Event` is a protocol-visible statement that the broker emits to describe a meaningful case change caused by command processing. Events are not the broker's complete internal event stream. They are the subset of state changes that matter at the User Agent boundary. Events carry references to the relevant entities and, where required for meaningful presentation, embed the changed entities inline so the User Agent can render the result without first calling `Get State`.

### 6.5 Pending Broker Action

A `PendingBrokerAction` is a typed broker-issued action that is currently awaiting resolution and that may block further progression of the case. In the conceptual model this corresponds to an open broker-user interaction (the whitepaper term is `CaseInteraction`) from the User Agent's perspective. Each pending broker action defines what the broker needs, whether the action is regulated, what response form is expected, and which protocol entities the action concerns.

### 6.6 Broker Action Resolution

A `BrokerActionResolution` is the structured protocol record of how the User Agent responded to a pending broker action on behalf of the user. It captures the formal outcome of the action, not just conversational context. Depending on the action's response expectation, the resolution may take forms such as acknowledgement, grant or refusal, affirmation or denial, authorisation or refusal, confirmation of direction, or provision of requested information through `Provide`.

### 6.7 Entity Reference

An `EntityReference` is a transport-neutral pointer to a protocol-defined entity. It identifies the target entity sufficiently for a User Agent or broker to correlate events, state, selections, and action subjects without requiring the entity to be repeated everywhere in full. At minimum, an entity reference is expected to include at least an identifier and an entity type discriminator, with optional embedded summary data where doing so improves interoperability or presentation clarity.

### 6.8 Contribution

A `Contribution` is the structured material that the User Agent provides into a case through `Open Case` or `Provide`. A contribution may contain one or more financial facts, party attributes, financial goals, evidence references, and optional opaque transcript material. The broker MAY store opaque transcript fields for audit or user replay purposes, but MUST NOT rely on them as decision-bearing inputs. Protocol meaning comes from the structured contributed data, not from natural-language text.

### 6.9 Selection

A `Selection` is the structured expression of the user's decision to proceed with a specific protocol-defined entity, typically a `FinancialPlan` or `ProductOffer`. A selection is semantically distinct from a contribution: it does not supply facts about the world, but rather records a commitment with lifecycle significance. A selection may trigger downstream broker work, including the issuance of new broker actions that must be resolved before the selected path can continue.

### 6.10 Withdrawal

A `Withdrawal` is the user's instruction, conveyed by the User Agent, to terminate the case before normal completion. It is a lifecycle decision rather than a data update. Processing a withdrawal moves the case toward a terminal withdrawn outcome and may cause the broker to emit terminal events or a terminal pending broker action, such as a `case_outcome`, depending on the case state and binding-independent broker behaviour.

## 7. Canonical Operations

This section defines the canonical operation set.

Unless a later section states otherwise, mutating operations are evaluated against the current case state at the time the broker receives them and return a `CommandResponse`. Where an operation payload refers to a broker-defined type such as a financial fact type, party attribute type, goal type, product feature, product category, or evidence type, the payload SHOULD use identifiers taken from the broker's published resources rather than free-form labels.

### 7.1 Open Case

`Open Case` creates a new case and optionally seeds it with initial structured material known at the start of the engagement.

The semantic purpose of `Open Case` is to establish the broker-managed lifecycle container in which all later protocol activity will occur. It is the only operation that creates a new case identifier.

At minimum, the minimum input shape for `Open Case` MUST allow the broker to create a case even when the User Agent has little or no structured information yet. The operation MAY therefore be invoked with no initial contribution material, or with an initial contribution containing any combination of:

- financial facts
- party attributes
- financial goals
- evidence references
- optional opaque transcript or conversational context material for audit.

`Open Case` MAY accept broker-specific opening context beyond the shared protocol fields, but such extensions MUST NOT change the transport-neutral semantics of case creation.

If initial contribution material is supplied, the broker MUST process it according to the same routing and validation semantics used by `Provide`. Facts and attributes are routed according to their declared structure and context. Goal constraints are attached to the relevant goal. Opaque transcript material MAY be stored, but MUST NOT be treated as decision-bearing input.

The effects of a successful `Open Case` are:

- creation of a new case identifier
- creation of an initial lifecycle state for the case
- recording of any initial contribution material that was accepted
- emission of any protocol-visible events caused by accepted initial material
- issuance of an initial pending broker action if the broker determines that an opening gate must be closed before progression continues.

The response to `Open Case` MUST be a `CommandResponse`. That response MUST allow the User Agent to learn the new case identifier and any immediately active pending broker action. If the broker requires an opening disclosure, consent, or other gate before further progression, the `CommandResponse` MUST surface it as the active pending broker action rather than relying on out-of-band instruction.

### 7.2 Provide

`Provide` submits structured contribution material into an existing case.

Its semantic purpose is to let the User Agent contribute new or corrected structured data as the user supplies it, without requiring the broker to drive each data-collection step proactively.

At minimum, a `Provide` request MUST target an existing case and MUST contain at least one structured contribution item. A single `Provide` call MAY carry any mixture of:

- financial facts
- party attributes
- financial goals
- evidence references
- optional opaque transcript material.

Structured data submitted through `Provide` MUST be expressed using protocol-defined structures plus broker-published vocabulary identifiers where applicable. The broker MUST validate that contributed items are structurally valid, refer to recognised broker vocabulary entries where required, and declare sufficient routing context for the broker to place them correctly in case state.

`Provide` creates or extends a contribution record in the case history. If a contributed item supersedes earlier contributed state, the broker MUST preserve the audit history of the earlier contribution while making the newer accepted value effective in the current case projection.

`Provide` has a special relationship to `information_request` pending broker actions. When the current blocking action is an `information_request`, `Provide` is the normal means by which the User Agent satisfies that request. The broker evaluates the newly provided structured data against the outstanding information need and, if the need is satisfied, resolves the gate through normal broker processing. If the contribution is incomplete or does not match the requested scope, the gate MAY remain active.

If a different kind of pending broker action is unresolved, `Provide` MUST be rejected as blocked by pending action. `Provide` is not a general bypass around broker-controlled gates.

The response to `Provide` MUST be a `CommandResponse` describing the protocol-visible effects of the accepted contribution, including any resulting state-change events and any newly active pending broker action.

### 7.3 Get State

`Get State` retrieves the broker's current projection of an existing case.

Its semantic purpose is to provide the User Agent with a full current picture of the case when a delta-oriented command response is not sufficient, for example after reconnect, after local state loss, or when the user asks to review everything currently known.

`Get State` is non-mutating. It MUST NOT create contributions, selections, broker action resolutions, events, or lifecycle transitions merely because it was called. It is a read operation over the broker's current protocol-visible case projection.

`Get State` MUST be callable even when a pending broker action is active, because the User Agent may need to inspect the current case and the active gate before deciding how to proceed.

The response to `Get State` MUST be a `FullStateResponse`. At minimum that response is expected to include:

- the case identifier
- current lifecycle status
- current protocol-visible entities and projections
- the currently active pending broker action, if any
- enough metadata to interpret freshness or ordering safely.

### 7.4 Select

`Select` records the user's decision to proceed with a specific selectable protocol entity in an existing case.

Its semantic purpose is to distinguish commitment from information supply. A selection is not a fact about the user's circumstances. It is a decision with lifecycle significance that may cause the broker to progress the case, perform further analysis, source offers, or raise a new gate.

`Select` MUST support, at minimum, selection of:

- a `FinancialPlan`
- a `ProductOffer`.

The selected target MUST be identified by entity reference. The referenced entity MUST exist in the current case state and MUST be selectable in the broker's current lifecycle position. A broker MAY support additional selectable entity types, but only if they are defined clearly enough for the User Agent to know what selection means.

`Select` MUST be rejected if:

- the referenced entity does not exist in the case
- the referenced entity is not selectable in the current case state
- a blocking pending broker action is unresolved.

The response to `Select` MUST be a `CommandResponse`. That response MAY include events indicating that the selection was accepted, that downstream case state changed, or that a new pending broker action has been raised before the selected path can continue.

### 7.5 Resolve Action

`Resolve Action` submits the user's formal response to a pending broker action.

Its semantic purpose is to capture explicit resolutions of broker-controlled gates in a structured form that the broker can act on safely and audit later.

`Resolve Action` MUST target a currently active pending broker action in the referenced case. The submitted resolution MUST match the action's declared `response_expectation`. At minimum, the allowed resolution patterns correspond to the protocol-fixed response expectations and include:

- acknowledgement
- grant or refuse
- affirm or deny
- authorise or refuse
- confirm direction.

`Resolve Action` is not the normal means of satisfying an `information_request`. Where the active pending broker action expects information provision, the User Agent SHOULD use `Provide` with the requested structured data. `Resolve Action` MUST NOT be used to pretend an information need has been met when the requested information has not been provided.

`Resolve Action` MUST be rejected if:

- there is no active pending broker action for the case
- the targeted action is not the currently resolvable action
- the supplied resolution form does not match the action's declared expectation
- the case is already terminal.

The response to `Resolve Action` MUST be a `CommandResponse`. That response MUST report the protocol-visible effects of the resolution, including any events caused by the broker's follow-on processing and any newly active pending broker action.

### 7.6 Withdraw

`Withdraw` communicates that the user wants to terminate the engagement.

Its semantic purpose is to let the User Agent end the case explicitly rather than attempting to simulate withdrawal through refusal, inactivity, or malformed responses to some other pending action.

`Withdraw` targets an existing case and carries the user's withdrawal decision. It MAY also carry optional opaque explanatory material for audit or user replay, but withdrawal semantics do not depend on the broker interpreting that text.

Unlike ordinary mutating operations, `Withdraw` SHOULD remain available even when another pending broker action is active. User withdrawal is a higher-order lifecycle decision and MUST NOT require the user to resolve an unrelated gate first.

On successful processing, `Withdraw` moves the case toward a withdrawn terminal outcome. At minimum this may be expressed directly through terminal events, through issuance of a terminal `case_outcome` pending broker action, or through both, provided that the transport-neutral lifecycle meaning remains clear.

The response to `Withdraw` MUST be a `CommandResponse`. After withdrawal has been processed, the case MUST be treated as terminal for further command progression except where a later section explicitly allows otherwise. `Get State` MAY still be used to retrieve the final case projection.

## 8. Shared Message Model

This section defines the transport-neutral message structures used by the protocol.

### 8.1 Request Envelope

This release uses a per-operation request type model rather than a universal transport-neutral request envelope.

Each canonical operation therefore has its own request shape whose required fields are determined by that operation's semantics. This keeps the protocol model close to the conceptual operation set and avoids introducing a wrapper object that exists only for stylistic consistency.

Bindings MAY add transport-specific framing where required by their mechanics, but such framing is not part of the transport-neutral protocol message model unless it changes protocol meaning.

Across operation request types, the following transport-neutral expectations apply:

- mutating requests identify the target case except for `Open Case`
- request fields that refer to broker-defined vocabularies SHOULD use identifiers from broker-published resources
- opaque transcript or narrative fields MAY be carried where the protocol explicitly allows them
- request payloads MUST remain decision-bearing through structured fields rather than relying on free text.

### 8.2 Command Response

`CommandResponse` is the shared success response returned by all mutating operations.

Its purpose is to report the protocol-visible result of a successfully processed command without forcing an immediate `Get State` call. It is a delta-oriented response, not a full case snapshot.

At minimum, `CommandResponse` SHOULD contain at least:

- `case_id`: the case to which the command result applies
- `events`: ordered array of protocol-visible events emitted by the command, which MAY be empty
- `pending_action`: the currently active pending broker action after command processing, or `null` if no gate is currently active
- `request_id`: optional correlation identifier echoing binding-level request correlation when available
- `sequence`: optional monotonically increasing case-local sequence marker if the implementation chooses to expose one
- `generated_at`: optional RFC `3339` timestamp indicating when the response was produced.

`events` in a `CommandResponse` MUST describe the broker's protocol-visible effects in command-result order. If the command changes state but does not create a user-meaningful delta beyond the current pending action, the `events` array MAY be empty.

`pending_action` in a `CommandResponse` is the post-command active gate, not merely a newly created action. This means the field tells the User Agent what gate, if any, is active after broker processing has settled for that command.

Errors are not represented as `CommandResponse`. They are handled through the protocol error model and binding-specific error expression.

### 8.3 Full State Response

`FullStateResponse` is the response returned by `Get State`.

Its purpose is to represent the broker's current protocol-visible projection of the case rather than the delta caused by one command.

At minimum, `FullStateResponse` SHOULD contain at least:

- `case_id`
- `case_status`
- `entities`: the current protocol-visible entity set or entity projection for the case
- `pending_action`: the currently active pending broker action, or `null`
- `last_sequence`: optional latest case-local sequence marker if exposed by the implementation
- `generated_at`: RFC `3339` timestamp for the projection
- `resource_versions`: optional mapping of resource kinds to versions if the broker wants to help the User Agent reason about which published vocabularies informed this state.

`FullStateResponse` MAY contain summary views and fully embedded entities together, provided the relationship between summaries, references, and embedded entities remains clear.

Unlike `CommandResponse`, `FullStateResponse` is not limited to the changes caused by one operation. It is the broker's current view of the case after all accepted mutations up to the point of retrieval.

## 9. Event Model

This section defines the initial event catalogue and the common structure shared by all protocol-visible events.

### 9.1 Event Structure

An `Event` is a transport-neutral record of a protocol-visible case change caused by command processing.

At minimum, every event SHOULD contain at least:

- `id`: stable event identifier
- `event_type`: protocol event type
- `case_id`: case to which the event belongs
- `occurred_at`: RFC `3339` timestamp for when the broker considers the event to have occurred
- `sequence`: case-local ordering marker sufficient to order events within a case
- `entity_refs`: zero or more references to entities materially involved in the event
- `entities`: zero or more inline embedded entities needed for meaningful client presentation.

`sequence` MUST be sufficient for a User Agent to preserve the broker's intended event order within a single `CommandResponse` and, where later responses are compared, within the same case.

`entity_refs` and `entities` work together:

- `entity_refs` identify the affected entities
- `entities` provide the inline data needed to present the event without necessarily calling `Get State`.

An implementation MAY embed only the entities that are necessary for client interpretation of that event. It need not embed the whole case projection in each event.

### 9.2 Initial Event Types

The event catalogue includes, at minimum, the following event types:

- `profile_updated`
- `goals_updated`
- `plans_ready`
- `plans_revised`
- `offers_ready`
- `offers_revised`
- `case_status_changed`.

The intended semantics of these event types are:

- `profile_updated`: financial profile facts or party attributes visible through the protocol projection have been added, corrected, superseded, or otherwise updated
- `goals_updated`: one or more financial goals have been recorded, changed, or removed in the current projection
- `plans_ready`: the broker has produced an initial set of plans, typically with associated assessments or recommendations where applicable
- `plans_revised`: previously produced plans have changed because upstream case inputs changed
- `offers_ready`: the broker has produced or received an initial set of product offers
- `offers_revised`: previously available offers have changed, been replaced, or been invalidated and refreshed
- `case_status_changed`: the case lifecycle status has changed.

In addition to the minimum set above, implementations MAY emit additional protocol-visible event types where needed for meaningful behaviour, provided those event types are clearly defined and do not conflict with the shared semantic core.

Where a pending broker action is issued, the active `pending_action` field in `CommandResponse` is normative for client behaviour. An implementation MAY also emit an event describing the issuance of that action, but such an event is optional in this release and MUST NOT be the only way the client learns that a gate is active.

## 10. Pending Broker Actions and Gates

This section defines the gate model that controls case progression.

### 10.1 Pending Broker Action Structure

A `PendingBrokerAction` is the transport-neutral representation of a currently unresolved broker action.

At minimum, a pending broker action SHOULD contain at least:

- `id`: stable action identifier
- `action_type`: one of the protocol action types
- `case_id`: the case to which the action belongs
- `issued_at`: RFC `3339` timestamp for when the action was issued
- `blocking`: boolean indicating whether the action blocks further progression
- `regulated`: boolean indicating whether the action carries verbatim handling obligations
- `content`: broker-authored content to be presented, where applicable
- `content_format`: optional content format indicator such as `text/plain`, `text/markdown`, or equivalent binding-compatible value
- `response_expectation`: the protocol-fixed response expectation for the action
- `subjects`: zero or more referenced subjects giving the action's domain context.

For this release, inline `content` is the default representation. Content references or externally dereferenced action payloads are deferred unless a later revision chooses to introduce them explicitly.

Action-type-specific fields MAY also be present where required by the action semantics, including:

- `information_scope` for `information_request`
- `purpose` and `consent_required` for `consent`
- `consequences`, `destination_url`, and `destination_name` for `instruction`
- `target_case_status` for `case_outcome`.

### 10.2 Action Types

The action-type catalogue is:

- `information_request`
- `disclosure`
- `consent`
- `declaration`
- `instruction`
- `case_outcome`.

The expected response forms for these action types are:

- `information_request` -> `provide_information`
- `disclosure` -> usually `acknowledge`
- `consent` -> `grant_or_refuse`
- `declaration` -> `affirm_or_deny`
- `instruction` -> usually `authorise_or_refuse` or `confirm_direction`, depending on the instruction semantics
- `case_outcome` -> usually `acknowledge` or `none`.

These are default expectations for interoperability. Individual action instances remain authoritative through their explicit `response_expectation` field.

### 10.2.1 Subject Roles

The subject-role catalogue is:

- `primary`
- `selected_option`
- `related_goal`
- `related_plan`
- `related_offer`
- `supporting_entity`.

These roles give the User Agent lightweight domain context for each action subject without requiring action-type-specific schema families. The minimum intent is:

- `primary`: the main entity the action is about
- `selected_option`: the option the user has chosen and is now being asked to review, confirm, or otherwise respond to
- `related_goal`: a goal that materially frames the action
- `related_plan`: a plan related to the action but not itself the primary selected option
- `related_offer`: an offer related to the action but not itself the primary selected option
- `supporting_entity`: another protocol-visible entity included for supporting context.

### 10.3 Gate Rules

The gate model is:

- when a blocking pending broker action is active, normal case progression is paused until that action is resolved or superseded by broker processing
- `Get State` remains allowed while a gate is active
- `Resolve Action` remains allowed for the active action when the submitted resolution matches the action's expectation
- `Provide` remains allowed only when the active blocking action is an `information_request`
- `Withdraw` remains allowed as an overriding lifecycle decision
- `Open Case` is not applicable once a case exists.

If a blocking pending action uses `response_expectation = none`, the gate is an explicit broker-controlled pause that does not require `Resolve Action` from the User Agent. In that case, the broker MUST clear or supersede the action through broker processing without requiring a user-side resolution payload.

When a blocking pending action is active, `Select` MUST be rejected. `Provide` MUST also be rejected unless it is being used to satisfy an active `information_request`.

For `information_request`, the special role of `Provide` is normative. The User Agent resolves the gate by supplying the requested structured data in the requested contexts. The broker then determines whether the outstanding information need has been satisfied. Partial satisfaction MAY leave the gate active.

When an action is refused, denied, or otherwise resolved negatively, the broker MUST determine the next protocol-visible case state explicitly. Depending on the action type and broker rules, that may result in:

- continued case progression along an alternative path
- issuance of a different pending broker action
- a case-status change
- a terminal `case_outcome`.

The broker MUST NOT leave the User Agent to infer the effect of a negative resolution from absence of change alone. The result must be made visible through the resulting `CommandResponse`, including events, pending action, or both.

## 11. Entities and References

This section defines the initial representation strategy for protocol entities.

### 11.1 Entity Representation Approach

Protocol-visible entities are represented as typed objects with shared metadata plus type-specific payload fields.

At the transport-neutral level, an embedded entity SHOULD contain at least:

- `id`
- `entity_type`
- any type-specific fields needed to make the entity meaningful to a User Agent.

`entity_type` identifies what kind of protocol entity the object represents. The type-specific fields then carry the semantics of that entity instance.

The protocol uses both embedded entities and entity references:

- embedded entities are used when the client needs enough inline data to present or interpret the result immediately
- entity references are used when identity and correlation are sufficient, or when repeating the full payload would be unnecessary.

An implementation MAY include the same entity both by reference and inline in the same response. Where that happens, the `id` and `entity_type` values MUST agree.

For interoperability, an embedded entity MUST contain enough information that a general-purpose User Agent can:

- identify what the entity is
- distinguish it from other entities in the case
- understand its immediate role in the current event, action, or case state
- present it in a meaningful user-facing way without relying on undocumented broker-specific interpretation rules.

### 11.2 Entity References

An `EntityReference` is the canonical identity pointer used across events, pending broker actions, selections, and state responses.

At minimum, an entity reference SHOULD contain:

- `id`
- `entity_type`.

It MAY also contain lightweight convenience fields such as:

- `label`
- `summary`.

Identifier requirements for references are:

- `id` MUST be stable within the case and broker surface for the life of the referenced entity
- `entity_type` MUST be sufficient to disambiguate the semantics of the referenced object
- a reference MUST NOT rely on human-readable labels alone.

Embedding rules in this release are:

- events SHOULD use `entity_refs` for correlation and MAY use inline entities for presentation
- pending broker actions SHOULD use references in `subjects` and MAY include short labels or summaries
- full state responses MAY embed the current protocol-visible entity set directly, with or without a parallel index of references
- selection requests MUST identify the selected entity by reference rather than by copied inline entity payload.

### 11.3 Initial Entity Set

For this release, the protocol MUST represent at least the following entity types concretely enough for interoperability:

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

This set reflects the minimum case material a User Agent needs to understand:

- user-supplied circumstances and goals
- broker-produced plans and offers
- advised or assessed outputs where the broker provides them
- contribution and response artefacts needed for audit-aware interpretation.

The protocol MAY expose additional entity types in this release, but the shared semantic core above is the minimum target for this document set.

## 12. Broker Vocabulary and Published Resources

The protocol makes a deliberate distinction between structure and vocabulary.

The protocol defines the structural model that every conforming broker shares, including operation semantics, event types, broker action types, lifecycle concepts, identifier rules, response expectations, and the common shape of protocol entities. The broker defines the business-specific vocabularies that populate that structure, such as which financial fact types it recognises, which party attributes it accepts, which goal types it supports, and which product features it uses when describing plans and offers.

For this release, broker-published resources are part of the protocol surface. They are not optional explanatory extras. A conforming User Agent is expected to read them and use them to map user intent into the broker's structured types. A conforming broker is expected to publish them in machine-readable form through each binding it exposes.

### 12.1 Published Resource Model

A published broker resource is a read-only protocol artefact that a User Agent can retrieve independently of any specific case. Published resources exist to describe broker-defined vocabularies and related semantic catalogues that the User Agent needs in order to participate correctly.

In transport-neutral terms, a published resource has:

- a stable resource identifier within the broker's published surface
- a resource kind indicating what catalogue it contains
- version or freshness metadata sufficient for caching and refresh behaviour
- a machine-readable payload whose fields have transport-neutral meaning.

The transport-neutral specification defines what resources mean. The REST and MCP bindings define how those resources are discovered, addressed, and transferred.

### 12.2 Mandatory Broker Resources

A broker implementing the full protocol model for this release MUST publish, at minimum, the following machine-readable resources:

- `financial_fact_types`
- `party_attribute_types`
- `goal_types`
- `financial_product_features`
- `financial_product_categories`
- `evidence_types`

These resources are mandatory because they supply semantics that are broker-defined rather than protocol-fixed:

- `financial_fact_types` tells the User Agent which financial circumstance types the broker recognises and how to classify user-provided facts.
- `party_attribute_types` tells the User Agent which personal or organisational attributes the broker accepts and how those attributes are structured.
- `goal_types` tells the User Agent which goal categories the broker supports and how to distinguish them.
- `financial_product_features` tells the User Agent how plan constraints, product specifications, offers, and recommendations name and type product features such as APR, term length, fees, or repayment frequency.
- `financial_product_categories` tells the User Agent how the broker classifies product families when a plan or requirement is category-based rather than product-specific.
- `evidence_types` tells the User Agent how supporting evidence should be classified when evidence is provided or referenced.

A broker MAY publish additional resources beyond this minimum set, for example product information requirement catalogues, lender-specific vocabularies, or richer planning taxonomies. Additional resources MUST NOT redefine the transport-neutral meaning of protocol-fixed structures.

### 12.3 Minimum Vocabulary Entry Requirements

Every entry in a broker-published mandatory resource SHOULD include enough information for an AI-capable User Agent to apply the vocabulary through comprehension rather than brittle string matching. At minimum, that means each entry SHOULD include at least:

- a stable type or catalogue entry identifier
- a human-readable label
- a rich description stating what the entry means and, where useful, what it excludes
- structural typing information needed to form valid protocol payloads
- natural-language examples showing how users may express the concept conversationally.

In addition, some resource kinds require structural hints:

- `financial_fact_types` entries SHOULD identify their expected value type, supported operators, and permitted information contexts.
- `party_attribute_types` entries SHOULD identify their expected value structure.
- `goal_types` entries SHOULD identify common usage patterns and the kinds of desired-change constraints typically associated with the goal.
- `financial_product_features` entries SHOULD identify their feature value type and any important presentational or comparison semantics.
- `financial_product_categories` entries SHOULD identify parent-child relationships where the catalogue is hierarchical.
- `evidence_types` entries SHOULD identify the kinds of facts or attributes they commonly substantiate where that distinction matters to broker behaviour.

### 12.4 Protocol-Fixed Classifications Versus Broker-Defined Vocabularies

Not every controlled list in the conceptual model is a broker-published vocabulary resource.

For this release, the following categories are protocol-fixed structural classifications rather than broker-defined vocabularies:

- value operators
- information contexts
- broker action types
- response expectations
- event types
- case statuses
- subject roles.

Bindings MAY expose these classifications as documentation or convenience resources, but they are not broker-specific resources and their meanings MUST NOT vary by broker.

### 12.5 Relationship to Broker Instructions

Broker-published vocabulary resources are distinct from broker instructions, prompts, or other client-guidance material.

Vocabulary resources define broker-specific domain meanings and type catalogues. Broker instructions define how a client should behave when using the protocol, including regulated-content handling and other behavioural constraints. Both are important for AI-capable User Agents, but they are not the same artefact type and SHOULD remain conceptually separate in this document set.

## 13. Identifiers, Correlation, and Versioning

This section defines the initial technical rules for identifiers, correlation, and sequencing metadata.

### 13.1 Identifier Classes

The protocol uses distinct identifier classes for:

- cases
- entities
- pending broker actions
- events
- request correlation.

These identifier classes MAY share a common underlying identifier family, but they are semantically distinct and MUST be treated as such in the specification and bindings.

### 13.2 Case Identifiers

Each case MUST have one stable case identifier assigned at case creation. That identifier remains stable for the lifetime of the case and is the primary correlation handle across all later operations and responses.

### 13.3 Entity Identifiers

Each protocol-visible entity SHOULD have a stable identifier within the broker surface. The same identifier MUST be used consistently wherever that entity is referenced or embedded in protocol payloads.

This release does not require globally portable identifiers across different brokers. Stability and unambiguous identity within one broker implementation are the immediate requirement.

### 13.4 Action and Event Identifiers

Each pending broker action and each event SHOULD have its own stable identifier. These identifiers support audit correlation, replay discussion, and binding-independent tracking of what happened.

Action identifiers and event identifiers MUST NOT be overloaded with case identifiers or entity identifiers.

### 13.5 Request Correlation

Bindings MAY expose request-correlation identifiers such as `request_id`. Where present, correlation identifiers help the User Agent map a broker response back to a specific request or local interaction attempt.

Request-correlation identifiers are transport-adjacent but semantically useful enough to permit them in the transport-neutral response model as optional metadata.

### 13.6 Sequence and Version Markers

This release allows, but does not require, case-local sequencing markers such as:

- `sequence` on `CommandResponse` and `Event`
- `last_sequence` on `FullStateResponse`.

If a broker exposes sequence markers, they MUST be monotonic within a case and MUST preserve the broker's intended ordering of protocol-visible changes.

Published resources MAY also expose version markers such as `version` on `PublishedResource`. These are not case versions. They are resource-freshness indicators and MUST be kept conceptually separate from case-local sequencing.

### 13.7 Provisional Aspects

The following remain intentionally provisional in this release:

- whether one UUID version is mandated across all identifier classes
- whether sequence markers become required rather than optional
- whether optimistic concurrency tokens are introduced later
- whether any cross-broker or registry-scoped identifier rules are needed in future versions.

## 14. Lifecycle and Progression Rules

This section defines the lifecycle model for a case.

### 14.1 Initial State

After successful `Open Case`, a case enters an active open state. At minimum this corresponds to `case_status = open` unless the broker immediately determines a different visible outcome, which SHOULD be rare and explicit.

### 14.2 Lifecycle Status Set

The initial lifecycle status set is:

- `open`
- `transferred`
- `withdrawn`
- `declined`
- `abandoned`
- `expired`.

`open` is the non-terminal working state. The others are terminal statuses for this release.

### 14.3 Progression Triggers

Case progression is triggered by successful processing of:

- `Open Case`
- `Provide`
- `Select`
- `Resolve Action`
- `Withdraw`.

`Get State` does not trigger progression.

Progression MAY involve:

- state enrichment without status change
- production or revision of plans
- production or revision of offers
- issuance or resolution of a pending broker action
- lifecycle status change.

### 14.4 Progression Blocking

When a blocking pending broker action is active, normal progression is paused. This means the case remains live, but certain commands cannot advance it until the gate is resolved or superseded.

The detailed operational effect of a gate is defined in `§10.3 Gate Rules`. The lifecycle rule is that a blocked case stays in its current status until broker processing explicitly changes that status.

### 14.5 Withdrawal Semantics

Withdrawal is a terminal lifecycle decision initiated by the user through the User Agent. Once withdrawal is accepted, the case moves toward `withdrawn` and further normal command progression stops.

The broker MAY express that terminal state through:

- immediate `case_status_changed` signalling
- a terminal `case_outcome`
- both.

### 14.6 Terminal Outcome Handling

When a case reaches a terminal status, the broker MUST make that fact visible through protocol-visible state, events, or both.

After a terminal status is reached:

- `Get State` MAY continue to be called
- mutating operations other than any explicitly allowed terminal-state mechanics MUST be rejected as invalid for current state.

The protocol SHOULD make terminal outcomes explicit rather than relying on inactivity or silence to imply closure.

## 15. Error Model

This section defines the initial transport-neutral error model independently of transport mechanics.

### 15.1 Error Categories

At minimum, the protocol error model distinguishes:

- `malformed_request`
- `unauthenticated`
- `unauthorised`
- `invalid_reference`
- `invalid_operation_for_state`
- `blocked_by_pending_action`
- `trust_level_insufficient`
- `unknown_or_unsupported_value`
- `content_integrity_failure`
- `stale_or_superseded_action`
- `concurrency_or_sequence_conflict`
- `internal_processing_failure`.

### 15.2 Category Meanings

The intended meanings are:

- `malformed_request`: the request payload is structurally invalid, missing required fields, or otherwise not processable as the intended operation
- `unauthenticated`: the request requires authentication, but no valid participant, account-bound user, guest authority, or equivalent credential was established
- `unauthorised`: authentication succeeded, but the authenticated participant, user authority, guest authority, or credential is not authorised for the requested case, resource, operation, or scope
- `invalid_reference`: a referenced case, entity, or action identifier does not exist or is not valid in the referenced scope
- `invalid_operation_for_state`: the operation is recognised, but it is not valid given the current case lifecycle state
- `blocked_by_pending_action`: the operation would otherwise be valid, but an active blocking pending broker action prevents it
- `trust_level_insufficient`: the authenticated participant is known and may have some protocol access, but the broker does not trust that participant to perform the requested operation or resolve the requested action type directly
- `unknown_or_unsupported_value`: the request uses a value the broker does not recognise or does not support, including unknown broker-vocabulary identifiers
- `content_integrity_failure`: broker-authored content, regulated content, or a protected protocol payload failed an integrity, signing, channel-binding, or equivalent validation check
- `stale_or_superseded_action`: the request targets a broker action that is already resolved, expired, superseded, or no longer the current resolvable action for the case
- `concurrency_or_sequence_conflict`: the request conflicts with exposed sequencing or concurrency expectations
- `internal_processing_failure`: the broker failed while attempting to process an otherwise acceptable request.

The distinction between `unauthorised`, `invalid_reference`, and `trust_level_insufficient` is security-sensitive. A broker MAY deliberately avoid revealing whether a case or entity exists when the caller lacks authority to access it. In that situation, the broker may use the binding's normal non-disclosure pattern while still preserving the semantic distinction internally for audit and diagnostics.

### 15.3 Semantic Error Payload Expectations

Regardless of binding, an error response SHOULD preserve enough semantic information for the User Agent to respond correctly. At minimum, that means an error representation SHOULD convey at least:

- the error category
- a short human-readable description
- the relevant case identifier when applicable
- the relevant pending broker action identifier when the error is `blocked_by_pending_action`
- any field or identifier name that caused the failure where that can be stated safely
- authentication, authorisation, trust, or integrity failure context where that can be stated without leaking protected case information.

### 15.4 Binding Preservation Requirement

Bindings may express errors differently, for example through HTTP status plus problem details or through MCP tool failure structures. However, the semantic distinction between the error categories above MUST survive across bindings.

In particular, a User Agent must be able to distinguish:

- a structurally bad request from a valid-but-blocked request
- unauthenticated access from authenticated-but-unauthorised access
- an unknown identifier from an invalid lifecycle transition
- insufficient trust from ordinary case access denial
- stale broker action resolution from a general concurrency conflict
- a broker processing failure from a client-side value problem.

## 16. Binding Model

This section describes how transport bindings relate to the transport-neutral protocol.

The transport-neutral specification is the normative source of protocol meaning. A binding realises the protocol correctly only to the extent that it preserves the same operation semantics, response semantics, event semantics, gate behaviour, entity meanings, and error meanings.

For this release, REST and MCP are considered equivalent realisations of the same protocol when:

- they expose the same canonical operation set
- they preserve the same preconditions and gate restrictions
- they return materially equivalent `CommandResponse` and `FullStateResponse` semantics
- they surface the same pending broker action meaning after commands
- they expose materially equivalent broker-published resources
- they preserve the same transport-neutral error categories
- they deliver the same behavioural instruction content through transport-appropriate mechanisms.

Transport-specific concerns may vary where they do not change protocol meaning. Examples include endpoint paths, HTTP status codes, MCP prompt names, MCP tool namespacing, cache headers, or other mechanics that exist only because of the transport.

### 16.1 Shared Canonical Schemas

This document set defines a common set of transport-neutral canonical schemas for the protocol's semantic structures. These schemas are shared by the REST and MCP bindings so that both bindings realise the same underlying protocol rather than drift into transport-specific variants.

The shared canonical schema set covers the protocol structures whose fields affect protocol meaning, including:

- operation request types
- command responses
- full state responses
- events
- pending broker actions
- broker action resolutions
- entity references
- the initial set of core protocol entities
- broker-published resource payloads whose fields affect protocol meaning.

These canonical schemas are defined outside the binding documents. In this document set, they are described in [protocol-schema-appendix.md](protocol-schema-appendix.md).

If machine-readable schemas are published as part of the current release, they should also live in a transport-neutral location such as a dedicated schema folder alongside the appendix, rather than inside the REST or MCP binding.

### 16.2 Binding-Specific Structures

Not every JSON structure should be shared across bindings. Structures that
exist only because of the transport should remain binding-specific.

The REST binding may therefore define transport-specific structures such as:

- endpoint paths
- HTTP methods
- headers
- HTTP status mappings
- REST-specific envelope fields, if any are introduced.

The MCP binding may therefore define transport-specific structures such as:

- tool names
- tool descriptions
- MCP-specific tool input or output framing required by the protocol server
- any MCP-specific metadata that does not change protocol meaning.

The boundary rule for this document set is:

- if a field changes the meaning of the protocol, it belongs in the shared
  canonical schema set
- if a field exists only because of HTTP or MCP mechanics, it belongs in the
  relevant binding.

### 16.3 Behavioural Content Across Bindings

The protocol requires more than payload structure alone for AI-capable clients. Clients also need behavioural guidance about how to use the protocol correctly.

For this release, that behavioural content MUST be made available in both bindings:

- in REST, through the discovery document's `instructions_markdown`
- in MCP, through the broker instruction prompt and mirrored instruction resource.

The transport may change the delivery mechanism, but not the underlying behavioural meaning. A broker MUST NOT publish materially different protocol rules in one binding from those it publishes in the other.

## 17. Security Model

This section defines the initial transport-neutral security model for the protocol. It does not define a complete production security profile and does not choose specific authentication mechanisms for REST, MCP, or any future binding. Its role is to state the security properties that every binding must preserve.

The protocol's security model protects broker accountability. A broker must be able to know which protocol participant invoked an operation, decide whether that participant is allowed to act on the relevant case, preserve an evidence record of what occurred, and prevent regulated broker-controlled moments from being altered or bypassed.

### 17.1 Participant Identity and Authentication

The broker MUST authenticate the calling protocol participant before accepting any operation that exposes private case data, mutates case state, resolves a broker action, or otherwise affects a case lifecycle.

The authenticated participant identity MUST be established by the binding's authentication mechanism or trusted deployment context. It MUST NOT rely solely on a self-asserted field in a protocol request body.

For User Agent-mediated journeys, the authenticated participant identity may represent a User Agent operator, a User Agent platform, a specific registered client, or another binding-defined participant identity. The identity MUST be stable enough for broker policy, trust calibration, and audit correlation.

Authentication is not the same as trust. Authentication establishes which participant is invoking the protocol. Trust determines which operations, action types, and regulated moments the broker is willing to let that participant handle directly.

### 17.2 User Authority Models

Case access may be authorised through either an account-bound user authority or a guest case-continuation authority.

In the account-bound model, the broker recognises the user through an authenticated broker-side account or equivalent durable user identity. The broker binds the case to that user authority and MUST allow case-scoped operations only when the authenticated user authority is authorised for the case and the authenticated protocol participant is authorised for the requested operation.

In the guest model, the broker does not recognise the user as a durable account holder. The broker MAY still create and continue a case, but it MUST issue or maintain a case-continuation authority that proves continuity of access to that case. A guest case-continuation authority proves permission to continue a specific case; it does not prove the user's durable real-world identity.

A binding MAY implement a guest case-continuation authority using OAuth tokens, opaque bearer credentials, signed capabilities, server-side sessions, or another suitable mechanism. Where OAuth is used for a guest journey, the token subject represents a pseudonymous guest participant or guest session rather than a known broker user account. The broker MUST preserve that assurance distinction in policy and audit.

### 17.3 Case Access Control

A case identifier is an identifier, not an access credential. Knowledge of `case_id` alone MUST NOT authorise access to a case.

The broker MUST enforce case access control consistently across all case-scoped operations and retrievals. At minimum, the broker MUST evaluate:

- the authenticated protocol participant
- the account-bound user authority or guest case-continuation authority
- the target case
- the requested operation
- the current case lifecycle state
- any active pending broker action
- any trust restrictions that apply to the participant for the requested operation or action type.

Guest case-continuation authorities SHOULD be high entropy, case-scoped, operation-scoped where practical, expiry-bound, and revocable. Where the binding supports it, the authority SHOULD be bound to the authenticated protocol participant or client context so that a continuation credential issued for one participant cannot be reused by another.

Bindings MUST define how account-bound user authorities and guest case-continuation authorities are carried, refreshed, challenged, and revoked. Bindings SHOULD avoid placing continuation secrets in user-facing prose, model-visible tool results, transcripts, broker-authored content, or other fields whose protocol role is presentation or evidence rather than credential transport.

### 17.4 Trust-Calibrated Operation Capability

The broker MUST decide which operations and action types an authenticated participant is trusted to perform directly. This decision MAY vary by participant, case, operation, pending broker action type, regulated flag, user authority model, assurance level, or other broker policy inputs.

A participant that is authenticated but insufficiently trusted for a regulated broker action MUST NOT be allowed to resolve that action directly merely because it can invoke the protocol. The broker SHOULD instead use the broker-controlled surface pattern, normally by issuing an `instruction` pending broker action that directs the user to the broker's own surface for the regulated step.

For guest cases, the broker SHOULD apply more conservative capability policy than for account-bound cases unless it has another basis for equivalent assurance. The broker MAY require step-up to an account-bound user authority, identity verification, or broker-controlled surface before allowing high-risk actions, lender transfer, full data export, or direct resolution of regulated disclosures, consents, declarations, or instructions.

### 17.5 Regulated Content Integrity

Broker-authored content in a regulated pending broker action is part of the broker-controlled compliance surface. The protocol requires that such content remain identifiable and preservable across bindings.

When `regulated = true`, the User Agent MUST treat broker-authored `content` as content to be presented faithfully according to the action semantics and broker instructions. The User Agent MUST NOT alter, omit, paraphrase, or reframe regulated content in a way that changes its meaning or weakens the broker's required presentation.

Bindings MUST define how regulated broker-authored content is protected in transit and how implementations can detect, prevent, or audit alteration of that content. The exact cryptographic, signing, notarisation, or channel-integrity mechanism is binding-specific and remains provisional in this release.

### 17.6 Evidence, Privacy, and Transcript Handling

The broker's evidence record is a security control as well as a regulatory record. For accepted case-scoped operations, the broker SHOULD record enough information to reconstruct the protocol-visible interaction, including:

- authenticated protocol participant identity
- account-bound user authority or guest case-continuation authority class
- operation name
- target case identifier
- timestamp
- request correlation identifier where available
- accepted structured payload or accepted structured payload reference
- pending broker action issued or resolved
- resolution outcome where applicable
- whether a regulated action was handled through the User Agent or through the broker-controlled surface.

The broker SHOULD protect evidence records, transcripts, and protocol payloads according to the sensitivity of the data they contain. Credit broking journeys commonly include personal, financial, and identity-related data.

Opaque transcript fields MAY be stored as evidence or used for user replay, complaint investigation, or benchmarking where appropriate. They MUST NOT be treated as decision-bearing input. A transcript may support audit, but the protocol meaning of a contribution or action resolution comes from the structured fields.

### 17.7 Replay, Duplicate Submission, and Stale Actions

The broker MUST prevent pending broker actions from being resolved after they are no longer current. A `Resolve Action` request targeting an already resolved, superseded, expired, or non-current action MUST be rejected.

The broker SHOULD treat duplicate mutating requests conservatively. This release does not require a transport-neutral idempotency-key mechanism, but bindings MAY define one. Where a binding or implementation supports idempotency, it MUST preserve the same protocol semantics as a single accepted operation and MUST NOT allow replay to bypass gates, repeat irreversible actions, or create contradictory evidence.

Sequence markers, action identifiers, request correlation identifiers, and binding-specific replay controls are complementary. None of them changes the rule that only the currently active resolvable pending broker action may be resolved.

### 17.8 Public Resources and Discovery

Broker-published resources, discovery documents, and behavioural instructions may have a different security posture from case-scoped operations. A broker MAY make some or all of these resources public or broadly readable so that a User Agent can understand the broker's protocol surface before opening a case.

Public or unauthenticated access to discovery and vocabulary resources MUST NOT imply access to any case state, case-scoped operation, private user data, or regulated action resolution capability.

Bindings MUST distinguish clearly between public bootstrap material, authenticated participant-level material, and case-scoped material.

### 17.9 Binding Requirements

Each binding MUST define how it realises the security properties in this section. At minimum, a binding security profile should state:

- how protocol participants are authenticated
- how account-bound user authorities and guest case-continuation authorities are represented
- how credentials or authorities are bound to a case, operation, client, or session where applicable
- how scopes, permissions, or equivalent authorisation controls map to protocol operations
- how authentication and authorisation failures are represented
- how regulated broker-authored content integrity is protected or audited
- which resources, tools, endpoints, prompts, or discovery artefacts may be accessed without account-bound or guest case authority
- how credentials are kept out of presentation, transcript, and other model-facing or user-facing protocol content.

## 18. Open Questions

The remaining cross-cutting questions are:

- whether machine-readable JSON Schema artefacts will be published alongside the prose appendix in this release or only after one more refinement pass
- whether sequence markers should remain optional or become required for all conforming implementations
- how much idempotency machinery should be standardised, especially for REST mutating operations
- where the long-term boundary should sit between embedded entities and references in command and state responses
- whether any asynchronous event feed, callback, or push mechanism belongs in a later version of the protocol
- how resource discovery, versioning, and change notification should evolve beyond the current position
- which additional broker-published resources should be recommended, but not mandatory, beyond the initial mandatory set
- which regulated-content integrity mechanisms should become mandatory in future production security profiles
- whether guest case-continuation authorities should have a shared minimum assurance profile across bindings

## 19. Relationship to Companion Documents

- [protocol-goals-and-scope.md](protocol-goals-and-scope.md) defines the release intent and scope.
- [protocol-standards-and-conventions.md](protocol-standards-and-conventions.md) defines the standards and conventions used across this document set.
- [protocol-schema-appendix.md](protocol-schema-appendix.md) defines the shared transport-neutral schema catalogue for the current release.
- [protocol-rest-binding.md](protocol-rest-binding.md) defines the REST realisation of this specification.
- [protocol-mcp-binding.md](protocol-mcp-binding.md) defines the MCP realisation of this specification.
