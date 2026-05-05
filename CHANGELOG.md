# Changelog

All notable changes to the Agentic Credit Broking Protocol will be documented in this file.

## [0.4] - 2026-04-29

### Added

- First `v0.4` technical-specification review set.
- Release intent and scope note in `docs/protocol-goals-and-scope.md`.
- Standards baseline in `docs/protocol-standards-and-conventions.md`.
- Transport-neutral technical specification in `docs/protocol-technical-spec.md`.
- Canonical schema appendix in `docs/protocol-schema-appendix.md`.
- REST and MCP bindings in `docs/protocol-rest-binding.md` and `docs/protocol-mcp-binding.md`.
- Whitepaper version bump to `v0.4`.

### Changed

- Repository positioning and documentation now describe the repo as a `v0.4` protocol document set.

## [0.3] - 2026-03-30

### Added

- Initial public release of the whitepaper describing the Agentic Credit Broking Protocol.
- Protocol interaction model: UA-driven turns with broker-controlled gates.
- Six protocol operations: Open Case, Provide, Get State, Select, Resolve Action, Withdraw.
- Broker action types: Information Request, Disclosure, Consent, Declaration, Instruction, Case Outcome.
- Domain entity vocabulary: Financial Profile, Financial Fact, Financial Goal, Financial Plan, Product Offer, Recommendation, Suitability Assessment.
- Trust framework with progressive trust levels and broker-controlled surface fallback.
- Evidence model with transcript support.
- Complaint investigation and replay mechanism.
