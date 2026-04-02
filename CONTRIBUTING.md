# Contributing to the Agentic Credit Broking Protocol

Thank you for your interest in the Agentic Credit Broking Protocol. We are developing this protocol in the open and welcome participation from across the ecosystem — AI platforms, financial services firms, regulators, and anyone interested in how regulated financial services work in an agentic world.

## How to Participate

### Feedback on the Whitepaper

The protocol is at an early stage. The most valuable contributions right now are:

- **Conceptual feedback** — Does the interaction model make sense? Are the role boundaries right? Are there gaps in the trust framework?
- **Domain expertise** — Perspectives from credit broking, lending, financial regulation, AI safety, or protocol design.
- **Use cases** — Scenarios that would test or break the current design.

Open an [issue](https://github.com/clearscore/agentic-credit/issues/new/choose) using the appropriate template.

### Proposals

If you have a substantive proposal — a new mechanism, an alternative approach to a problem the protocol addresses, or an extension — open a Proposal issue. Include the problem you're solving and your proposed approach.

### Questions

If something in the whitepaper is unclear, open a Question issue.

## Language

This project uses British English spelling (e.g. "behaviour", "standardised", "licence" for the noun). CI enforces this via spell check.

## Pull Requests

All changes go through pull requests, including changes from maintainers. Direct pushes to `main` are not permitted.

When submitting a PR:

1. Fork the repository and create your branch from `main`.
2. Fill in the pull request template — what changed, why, and any open questions.
3. Keep changes focused. One logical change per PR.
4. For whitepaper changes, explain the reasoning — the "why" matters more than the "what."

PRs will be squash-merged to keep `main` history clean.

## Commit Messages

We use [Conventional Commits](https://www.conventionalcommits.org/) for squash-merge commit messages on `main`:

```text
docs: clarify trust model for unknown User Agents
feat: add broker action type for identity verification
fix: correct event type table in section 4.3
```

Your individual commits within a PR branch don't need to follow this convention — the squash merge message will.

## Code of Conduct

Be constructive, specific, and respectful. This is a protocol for regulated financial services — precision and clarity matter.

## License

By contributing, you agree that your contributions will be licensed under the [CC BY-SA 4.0](LICENSE) licence.
