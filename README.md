# Two-Thirds

A social deduction game on Arc where every move is a transaction and the chain
counts the votes.

Arc City runs on one Ledger. A page is final only when more than two-thirds of
the Sealers stamp it, the same rule Arc's validators follow. Hidden among the
Sealers are Forkers. One forged page stamped and the city cracks in half. Three
rounds with nothing stamped and the city freezes.

- [Story](docs/story.md): the world, the three endings, the cast
- [Rules](docs/rules.md): sides, rounds, slashing, the Daily Ledger
- [Design](docs/design.md): the look and feel
- [Architecture](docs/architecture.md): contract, server, wallets, fairness
- [Plan](PLAN.md): build order
- [Arc Studio prompt](ARC_STUDIO_PROMPT.md): the brief for building it

## Status

Design. Nothing is built or deployed yet.

## Built on

Arc (USDC gas, sub-second deterministic finality) and Circle developer-controlled
wallets.

## Fairness, honestly

The contract counts every stamp, enforces the two-thirds rule and checks every
role and the game seed. In this version the server holds player wallets and the
salts for hidden votes, so it could read votes during play. The outcome is still
auditable afterwards from the chain alone.
