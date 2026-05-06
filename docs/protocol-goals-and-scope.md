# Agentic Credit Broking Protocol v0.4 Goals and Scope

## Status

This document is the goals-and-scope note for `v0.4` of the Agentic Credit Broking Protocol. It captures the goals, scope, and intended deliverables for the `v0.4` technical-specification review set.

It is not itself the technical specification. Its role is to define what `v0.4` is trying to achieve and what artefacts it should contain.

## Purpose

`v0.4` introduces the first technical specification for the protocol. Its purpose is to turn the protocol from a conceptual whitepaper into a concrete technical artefact that is implementable in principle, reviewable by engineers, and testable across early prototypes.

The release should define the protocol clearly enough that two independent teams could plausibly build against it, while remaining explicitly incomplete and non-final.

## Goals

The primary goals are:

- Define the first technical specification of the protocol, expanding the whitepaper's conceptual model into a concrete protocol contract.
- Make the protocol implementable in principle by specifying operations, payload shapes, entity representations, identifiers, lifecycle expectations, and error handling at an initial technical level.
- Provide two parallel protocol realisations: one in REST and one in MCP, with equivalent semantics and aligned core concepts.
- Preserve the protocol's existing conceptual shape from the `v0.4` whitepaper while translating it into a form that can be exercised in software.
- Keep the release intentionally provisional: plausible, coherent, and testable, but explicitly incomplete and not yet suitable as a final production or conformance standard.
- Establish a foundation for subsequent versions to refine transport details, conformance rules, governance, security profiles, and production-hardening requirements.

## Non-Goals

This release should not attempt to do the following:

- Finalise the full production-ready specification.
- Freeze the protocol for backward-compatible implementation commitments.
- Define the full conformance framework, certification regime, or benchmark governance.
- Resolve all open questions around trust, registry governance, or operational supervision.
- Standardise every domain vocabulary and edge case needed for all lenders, brokers, or jurisdictions.
- Commit to one transport as normative over the other. The release should show equivalence, not final transport governance.

## Scope

The scope is to define the protocol at a level where its operational shape is technically explicit. That means specifying the core operations, the structure of requests and responses, the representation of key protocol entities, the handling of pending broker actions and events, and the minimum behavioural rules needed for an implementation to interoperate meaningfully.

The scope also includes two aligned transport realisations, one for REST and one for MCP, so that the same protocol semantics can be exercised through both interface styles.

The scope should stay intentionally narrow where necessary. This release does not need to solve every edge case, enumerate every possible domain value, or lock down all future compatibility constraints. It only needs to make the protocol real enough to build against and evaluate as a technical artefact.

## Core Deliverables

The release should ship as the first technical specification review set for the protocol. The review set should make the protocol concrete enough to read, discuss, prototype, and test, without claiming production completeness or final stability.

The expected deliverables are:

- A technical specification document that defines the protocol semantics independently of transport.
- A REST realisation that maps the protocol operations and entities onto HTTP endpoints, methods, request bodies, responses, and error structures.
- An MCP realisation that maps the same protocol semantics onto MCP tools, tool inputs, tool outputs, and event or state handling conventions.
- A first set of concrete schema representations for the main protocol structures, sufficient for prototype implementations.
- Worked examples covering at least one end-to-end case flow across both REST and MCP.
- An explicit status statement describing what is defined, what is provisional, and what remains intentionally incomplete.

## Minimum Contents of the Spec

The specification should, at minimum, define:

- The canonical operation set and the semantics of each operation.
- The shared response model, including events, pending actions, and full-state retrieval.
- The initial technical representation of core entities and references.
- Identifier rules and correlation expectations.
- Blocking and progression rules around broker actions and gates.
- Basic error categories and failure semantics.
- The relationship between the transport-neutral specification and the REST and MCP bindings.

## Supporting Representations

To make the release testable as a technical artefact, it should include supporting representations rather than prose alone. These should include:

- Machine-readable or near-machine-readable schemas for core payloads.
- Example payloads for each operation.
- A simple protocol flow or sequence representation for a typical case.
- A mapping table showing semantic equivalence between the transport-neutral model, REST, and MCP.

## Status Constraints

The release should state plainly that it is provisional. In practice, that means:

- Some entities and vocabularies may be partial.
- Some lifecycle and error details may be provisional.
- Security treatment is initial only; binding-specific authentication profiles, conformance, and governance treatment may be incomplete.
- Backward compatibility should not yet be assumed.
- Implementations built against it are for prototyping, testing, and specification refinement, not production commitment.

## Candidate Artefact List

The natural release structure implied by this draft is:

- `Technical Specification`
- `REST Binding`
- `MCP Binding`
- `Schema Appendix`
- `Worked examples embedded in the REST and MCP bindings`
- `Changelog entry for the release`

## Relationship to Existing Docs

This document sits between the `v0.4` whitepaper and the concrete `v0.4` technical-specification artefacts in this repository:

- [whitepaper.md](whitepaper.md) remains the conceptual and narrative baseline at `v0.4`.
- [protocol-standards-and-conventions.md](protocol-standards-and-conventions.md) defines cross-document standards and requirement-language conventions used by the review set.
- [protocol-technical-spec.md](protocol-technical-spec.md) defines the transport-neutral normative semantics targeted by this release.
- [protocol-schema-appendix.md](protocol-schema-appendix.md) defines the transport-neutral schema catalogue and field expectations.
- [protocol-rest-binding.md](protocol-rest-binding.md) and [protocol-mcp-binding.md](protocol-mcp-binding.md) define equivalent transport realisations and include aligned worked examples.

This file serves as the release-intent and scope note for the accompanying technical-specification review set.

## Semantic Equivalence Mapping (v0.4)

This release includes the following transport-equivalence mapping for the canonical operations:

| Transport-neutral operation | REST realisation | MCP realisation |
| --- | --- | --- |
| `open_case` | `POST {case_root_url}` | Tool `open_case` |
| `provide` | `POST {case_root_url}/{case_id}/provide` | Tool `provide` |
| `get_state` | `GET {case_root_url}/{case_id}` | Tool `get_state` |
| `select` | `POST {case_root_url}/{case_id}/select` | Tool `select` |
| `resolve_action` | `POST {case_root_url}/{case_id}/resolve-action` | Tool `resolve_action` |
| `withdraw` | `POST {case_root_url}/{case_id}/withdraw` | Tool `withdraw` |
