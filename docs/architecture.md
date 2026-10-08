# Architecture

How Two-Thirds is built. The story is in [story.md](story.md), the rules in
[rules.md](rules.md), the look in [design.md](design.md), the order of work in
[../PLAN.md](../PLAN.md).

## Verified facts this design rests on (checked 2026-10-08)

| Fact | Source | Consequence |
|---|---|---|
| Arc Privacy Sector is "on the roadmap and not yet available on Arc" | docs.arc.io/arc/concepts/opt-in-privacy | Hidden votes use commit-reveal, not confidential contracts. APS is the v2 upgrade. |
| Circle developer-controlled wallets support `ARC` and `ARC-TESTNET`, EOA and SCA | developers.circle.com/wallets/supported-blockchains | Players get a Circle wallet the server signs for. No popups, no approvals. |
| Gas Station on Arc | Not confirmed anywhere | Use EOA wallets topped up with a few cents of USDC by the operator. Do not depend on sponsorship. |
| The Circle CLI "agent wallet" is one wallet per logged-in email, for an AI agent paying x402 services | circle skill `use-agent-wallet` | Wrong tool for players in a browser. Developer-controlled wallets are the right one. |
| Arc mainnet chain id 5042, explorer explorer.arc.io | arc-trade-primitives README, live deployment | Mainnet target. |
| Native USDC 18 decimals vs 6 on the ERC-20 interface; 20 gwei fee floor; sends to address(0) revert | Agent notes on docs.arc.io, NOT yet checked against the EVM-differences page | Verify before mainnet. The contract holds no USDC, so the decimals trap only touches gas top-ups. |

## Provably fair, within limits

At game start the operator posts on chain:

- `seedCommit = keccak256(seed)`. Roles, balances and every page are derived from
  the seed.
- One `roleCommit` per player, `keccak256(gameId, player, role, roleSalt)`.

At game end the seed is revealed and the contract checks it against the
commitment. Anyone can re-derive every role, balance and page and confirm the
operator dealt the game it committed to.

What this does not give: hidden votes from the operator. The server holds every
player's wallet and every salt, so it could read sealed votes during play. The
contract still counts honestly and the outcome is still auditable afterwards.
Say this plainly in the game's About page. Arc Privacy Sector closes the gap when
it ships, and that upgrade is a story worth posting.

## Architecture

```
Browser (Next.js)
  |  Sign in with X  (handle goes on the share card)
  |  WebSocket: game state, chat, live chain feed
  v
Game server (Node, TypeScript)
  - lobby, matchmaking, bot fill
  - phase clock, the state machine of a round
  - deals pages from the seed
  - signs every move through Circle developer-controlled wallets
  - operator hot wallet: createGame, closeRound, slash, endGame, gas top-ups
  |                         |
  v                         v
Circle Wallets API      Arc mainnet (5042)
(one EOA per player,    TwoThirds.sol: membership, proposer order,
 ARC chain code)        commit-reveal, 2/3 count, role and seed checks
                            |
                            v
                        Indexer: reads contract events, measures time to
                        finality per move, feeds the live panel and the
                        replay page
Postgres: users, wallets, games, chat. Redis: phase timers, pub/sub.
```

### Why the contract is the referee

The server runs the clock and deals the cards, but it cannot decide a page. The
count, the threshold and the slashing check all run in `TwoThirds.sol`. A replay
page rebuilt only from contract events shows the same game the players saw. That
line, "rebuilt from the chain alone", is the one to show the Arc team.

### Moves per game

7 players, 8 heights, about 1.3 rounds a height: roughly 2 transactions per
player per round plus proposals, slashing and operator calls. Call it 150 to 300
transactions a game.

At the docs' 20 gwei floor and about 60k gas a move that is around $0.0012 a
move and $0.20 to $0.40 a game. A $25 operator float runs about 60 to 120 games.
Measure the real figure on testnet first.

### The one latency risk

The Circle API signs and submits asynchronously, so a move may take one to three
seconds to reach the chain even though Arc finalises in under one. The phase
windows above absorb that. The live panel shows two numbers per move: time from
click to submission, and time from submission to finality. The second number is
Arc's and it is the one that will impress.

If click-to-chain is regularly over three seconds, the fallback is a per-game
session key: the player's Circle wallet registers a throwaway key in
`createGame`, the key signs moves directly over RPC, and it is discarded at game
end. Measure before building this.

## Rules the build keeps

- No money stakes, no prizes in USDC, no buying advantage. Points and cards only.
  It stays a game and stays promotable in the Arc community.
- No real people, companies or validators as characters. BlackRock, Visa and the
  rest are not in the game. Invent citizens and Council members.
- Testnet first, then mainnet. Same code, one config switch.
- The contract never holds user funds.

## Open checks before mainnet

1. Arc EVM differences page: decimals, fee floor, address(0), blocklist reverts.
2. Circle API rate limits and click-to-chain latency on Arc, measured.
3. Gas cost per move, measured on testnet.
4. X OAuth app approved for Sign in with X.
