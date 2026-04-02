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

## Read the Whitepaper

The full protocol description is in [docs/whitepaper.md](docs/whitepaper.md). It covers:

1. **The interaction model** — the UA drives the interaction; the broker controls the gates
2. **Protocol operations** — six operations through which UAs participate in a credit broking journey
3. **Events** — typed, structured descriptions of what the broker did in response to a UA operation
4. **Broker action types** — disclosures, consents, declarations, instructions, information requests, and case outcomes
5. **A domain entity vocabulary** — standardised types for financial entities that make up the shared case state
6. **A trust framework** — progressive trust calibrated per operation, with broker-controlled interaction as the designed fallback
7. **Benchmarking** — compliance testing and certification for User Agents

## Status

This protocol is in early development. The whitepaper (v0.3) describes the problem, the roles, the interaction model, and the trust framework. Formal specifications, schemas, and reference implementations will follow.

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
