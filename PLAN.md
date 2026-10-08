# Plan

Build order for Two-Thirds. Each stage ends in something that runs. Testnet
(`ARC-TESTNET`) until stage 7; mainnet (`ARC`, chain 5042) is a config switch.

Read first: [docs/story.md](docs/story.md), [docs/rules.md](docs/rules.md),
[docs/design.md](docs/design.md), [docs/architecture.md](docs/architecture.md).

## Decisions taken (change here if needed)

1. Names: Arc City, the Ledger, Sealers, Forkers, the Crack, the Freeze, Finality.
2. The Daily Ledger ships in v1. It is the main reason people come back.
3. Sealers are in the Zerun character family, same design system.
4. Hidden votes use commit-reveal. Arc Privacy Sector is not live yet (checked
   2026-10-08); moving to it is a later stage.
5. Player wallets are Circle developer-controlled EOAs, signed by the server.
   No popups. Gas is topped up in USDC by the operator, not Gas Station.
6. No money stakes or prizes, ever.

## Stage 0. Verify the ground

- Check Arc's EVM differences page: native USDC 18 vs ERC-20 6 decimals, the
  minimum fee, sends to address(0), blocklist reverts.
- Create a Circle developer-controlled wallet on `ARC-TESTNET`, fund it from the
  faucet, send one transaction through the API, and record click-to-chain time.
- Done when: all four facts are confirmed or corrected in docs/architecture.md,
  and one measured Circle-signed transaction exists on testnet.

## Stage 1. The contract

- `TwoThirds.sol` as specified in ARC_STUDIO_PROMPT.md, with Foundry tests and a
  fuzz test on the threshold.
- Deploy to `ARC-TESTNET`.
- Done when: every test passes and a scripted 7-player game runs end to end on
  testnet from a script, with every move a transaction.

## Stage 2. The server

- Room state machine and phase clock, deterministic dealing from a seed, role and
  seed commitments, per-wallet move queue, operator calls, gas top-ups with a
  daily cap.
- Rule-based bots.
- Done when: a bots-only game plays itself on testnet and the seed check passes.

## Stage 3. The indexer and chain feed

- Read contract events, store them, measure click-to-submission and
  submission-to-final per move, push to clients.
- Done when: every move in a bots-only game appears in the feed with both times.

## Stage 4. The table

- Sign in with X (guest fallback), lobby, the game table, the page, stamps,
  threshold meter, private balances drawer, chat, chain feed, mobile layout.
- Zerun design system per docs/design.md.
- Done when: one person plays a full 7-seat game against bots on a phone.

## Stage 5. Endings, cards and replay

- Finality, the Crack, the Freeze animations.
- End screen with roles revealed and the seed check.
- Share card image and one-click post to X.
- Replay page rebuilt only from chain events plus the published seed.
- Done when: a finished game produces a card, and its replay matches the game.

## Stage 6. The Daily Ledger

- One seed per day, same deal for everyone against bots, the emoji grid share,
  a daily leaderboard.
- Done when: two people on different devices get the same deal and different
  grids.

## Stage 7. Mainnet

- Measure gas per move and cost per game on testnet; size the operator float.
- Deploy the contract to Arc mainnet, verify the deployed bytecode matches the
  source, switch config.
- Done when: a full game runs on mainnet and its replay page works.

## After launch, one at a time (each is a post on X)

- Titles and feats, seal designs unlocked by feats.
- LLM bots that talk and bluff.
- Arc House clubs, Council Night, club leaderboard.
- Epochs with one new role each: Auditor, Light Client, Double Agent.
- Arc Privacy Sector for truly hidden votes, once it ships.
