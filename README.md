# Agentic Credit Broking Protocol

The Agentic Credit Broking Protocol enables credit broking journeys to occur through user-controlled AI assistants and applications while preserving regulatory accountability.

It separates three concerns that today are bundled together:

- **Interaction mediation** — performed by a User Agent (an AI assistant, financial app, or any digital interface)
- **Regulatory interpretation and responsibility** — performed by a Credit Broker
- **Credit decision-making** — performed by a Lender

The core insight: **the system that mediates the conversation need not be the system that carries regulatory responsibility.** AI assistants can participate in regulated credit journeys without understanding regulation. They gather information, present content, and relay decisions. Brokers interpret the interaction, determine whether obligations have been met, and accept responsibility for that judgement.

## Why This Exists

AI assistants are changing how consumers begin financial journeys. The regulated firm no longer controls the first interaction surface. Without a protocol, these journeys are opaque — to the broker, to the lender, and to any regulator who later needs to understand what occurred.

The protocol allows interaction and responsibility to travel separately. A User Agent can mediate a complete credit journey without becoming a regulated entity. The broker retains regulatory control and gains an evidence trail, even though the conversation happened elsewhere.

## Read the Document Set

The current protocol version is `v0.4`. The repository document set for `v0.4` includes:

1. [docs/whitepaper.md](docs/whitepaper.md) — the conceptual narrative: roles, protocol model, trust model, and benchmarking approach.
2. [docs/protocol-goals-and-scope.md](docs/protocol-goals-and-scope.md) — `v0.4` release intent, scope boundaries, deliverables, and status constraints for the technical-specification review set.
3. [docs/protocol-standards-and-conventions.md](docs/protocol-standards-and-conventions.md) — shared standards, requirement-language conventions, naming rules, and cross-document editorial constraints.
4. [docs/protocol-technical-spec.md](docs/protocol-technical-spec.md) — transport-neutral semantics for operations, gates, entities, lifecycle, errors, and security model.
5. [docs/protocol-schema-appendix.md](docs/protocol-schema-appendix.md) — canonical schema catalogue, field expectations, and transport-neutral structure boundaries.
6. [docs/protocol-rest-binding.md](docs/protocol-rest-binding.md) — HTTP/JSON realisation of the protocol, including discovery, endpoint mapping, and worked REST flow.
7. [docs/protocol-mcp-binding.md](docs/protocol-mcp-binding.md) — MCP tool/resource/prompt realisation of the same semantics, with parallel worked MCP flow.

## Status

This protocol is in early development. `v0.4` is the current version in this repository, comprising the whitepaper and the first technical-specification review set. The `v0.4` material is concrete enough for technical review and prototyping, but it is still provisional and should not be treated as a production-stable standard.

We are developing this protocol in the open and welcome feedback.

## Local Development

Three CI checks run on every pull request. To run them locally:

```bash
npx markdownlint-cli2 "**/*.md"     # Markdown lint
npx cspell "**/*.md"                 # Spell check (British English)

# lychee (link check) requires a separate install — see https://lychee.cli.rs/installation/
lychee --no-progress "**/*.md"
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to participate.

## License

This work is licensed under [Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](LICENSE).

## About

The Agentic Credit Broking Protocol is developed by [ClearScore Group](https://www.clearscoregroup.com/) - The Global Leader in Financial Marketplaces.
