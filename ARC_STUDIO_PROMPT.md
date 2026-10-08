Build "Two-Thirds", a real-time social deduction game where every player move is a transaction on Arc. The rules of the game are the rules of Arc's BFT consensus: a page is final only when more than two-thirds of the living Sealers stamp it. Build it testnet-first (ARC-TESTNET) with one config switch to Arc mainnet (chain id 5042, explorer https://explorer.arc.io).

Before writing code, read README.md, PLAN.md and everything in docs/ in this repository. docs/story.md, docs/rules.md and docs/design.md are the source of truth for the world, the rules and the look; where this prompt and those files disagree, the files win. Work through PLAN.md stage by stage and stop at the end of each stage with its "Done when" check shown passing. In this prompt "Council" means the Sealers and "seal" means stamp.

## The game

Arc City keeps one Ledger. Players are the Council (validators). Some are secretly Forkers who want a forged page (a double spend) sealed into the Ledger.

Players: 5 to 10. Forkers = floor((n - 1) / 3). Everyone else is Honest.

Forkers win if a forged page is ever finalised (safety broken), or if 3 heights stall with nothing finalised (liveness broken).
Honest players win if 5 valid pages are finalised, or every Forker is slashed.

Each height, the server deals a page of 5 fictional transactions between citizens ("Ada pays Bola 40 USDC"). Every Honest player privately sees the true balances of 2 or 3 citizens. Forkers see all balances. A forged page contains one transaction that overspends a citizen's balance. An Honest proposer always proposes the valid page. A Forker proposer chooses between the valid page and a forged one.

Round phases (server clock, configurable):
1. Propose, 10 s. The proposer is living player index (height + round) mod livingCount. They pick a page; the page hash goes on chain.
2. Debate, 45 s. Off-chain text chat. Players show their private balance knowledge, honestly or not.
3. Seal, 15 s. Every living player commits keccak256(abi.encode(gameId, height, round, vote, salt, player)). vote is 1 = seal, 0 = reject.
4. Reveal, 5 s. Commits are revealed. A missing reveal counts as not sealing.
5. Close. The contract counts. If seals * 3 > living * 2 the page is final and the next height starts at round 0. Otherwise the same height moves to the next round with the next proposer. A failed round on a valid page adds one stall. A rejected forged page shows a "Forgery caught" moment and adds no stall.

After every second finalised height, a Slashing vote: each living player commits a target (commit-reveal, same pattern). The plurality target is slashed; ties slash nobody. The operator reveals the slashed player's role and the contract checks it against the role commitment. Slashed players become spectators. Thresholds recompute from the living count.

Bots fill empty seats so a single visitor can play at once. Bots are clearly labelled. v1 bots follow simple rules: Honest bots reject pages that contradict balances they hold and claim their real balances in chat with short canned lines; Forker bots sometimes propose forgeries and sometimes lie about balances. Bots use operator-managed Circle wallets like everyone else, so their moves are on chain too.

## Smart contract: TwoThirds.sol (Solidity, Foundry)

The contract is the referee. The server runs the clock and deals pages; the contract enforces membership, proposer order, commit-reveal, the 2/3 count, role checks and the seed check. It holds no funds and has no payable functions.

State per game: players, alive flags, operator, seedCommit, roleCommit per player, current height and round, finalisedCount, stallCount, phase, status.

Functions:
- createGame(bytes32 gameId, address[] players, bytes32 seedCommit, bytes32[] roleCommits) onlyOperator
- propose(bytes32 gameId, uint256 height, uint256 round, bytes32 pageHash) only the scheduled proposer, only in the Propose phase
- commitSeal(gameId, height, round, bytes32 commitment) living players, Seal phase, once each
- revealSeal(gameId, height, round, uint8 vote, bytes32 salt) must match the commitment
- closeRound(gameId, height, round, bool forged, bytes32 pageSalt) onlyOperator, after reveals. Counts seals among living players with seals * 3 > living * 2. Records finalised or failed. If finalised and forged, the game ends with Forkers winning. Updates stallCount and finalisedCount and ends the game on 3 stalls or 5 finalised honest pages.
- commitSlash / revealSlash, same commit-reveal pattern with a target address
- resolveSlash(gameId, address target, uint8 role, bytes32 roleSalt) onlyOperator. Requires keccak256(abi.encode(gameId, target, role, roleSalt)) == roleCommit[target]. Marks target dead. Ends the game if every Forker is dead.
- endGame(gameId, bytes32 seed) onlyOperator. Requires keccak256(abi.encode(seed)) == seedCommit. Emits the final result.

Events for every state change, with enough data to rebuild the whole game from events alone: GameCreated, Proposed, SealCommitted, SealRevealed, RoundClosed(finalised, forged, seals, living), SlashCommitted, SlashRevealed, Slashed(target, role), GameEnded(winner, seed, finalisedCount, stallCount).

Use custom errors, no unbounded loops beyond the player list (max 10), and checks-effects order. Write Foundry tests for: proposer rotation; the threshold at every player count 5 to 10; a forged page finalising ends the game; three stalls end the game; five honest finals end the game; a reveal that does not match its commit reverts; a double commit reverts; a non-living player cannot act; resolveSlash rejects a wrong role or salt; endGame rejects a wrong seed. Include a fuzz test over random vote sets checking the count matches seals * 3 > living * 2.

## Wallets: Circle developer-controlled wallets

Every player and every bot gets a Circle developer-controlled EOA wallet on chain code ARC-TESTNET (ARC on mainnet). The server signs every move through the Circle Wallets API, so players never see a wallet popup or approve anything. Use the official Circle server SDK for developer-controlled wallets. The API key and entity secret live only in server environment variables, never in the browser, never in the repo.

Arc pays gas in USDC. Do not rely on Gas Station. When a wallet joins a game and holds less than 0.02 USDC, the operator wallet sends it 0.05 USDC for gas. Log every top-up. Keep a daily cap on total top-ups in config.

Arc specifics to respect (verify each against the Arc EVM differences docs before mainnet): native USDC uses 18 decimals while the USDC ERC-20 interface uses 6, so never mix the two in one calculation; there is a minimum maxFeePerGas (reported as 20 gwei); native sends to address(0) revert; a blocklisted transfer can consume gas without a receipt, so treat a missing receipt as a failure state, not a pending one.

## Server (Node 22, TypeScript)

- Game state machine per room with a phase clock in Redis. Phases advance on the clock, never on a client request.
- Deterministic dealing: roles, balances and every page are derived from a 32-byte random seed per game using a seeded PRNG. Post seedCommit and roleCommits in createGame. Publish the full seed and all page data at game end so anyone can re-derive and check every page hash.
- Move submission queue per wallet so nonces never collide. Retries with backoff. A move that misses its phase window is dropped and counts as not acting.
- Indexer that reads contract events, stores them, and measures for each move: click to submission, and submission to finalised block. Push both to clients.
- WebSocket for game state, chat and the live chain feed. Rate-limit chat. Strip links and markup from chat.
- Postgres for users, wallets, games, moves, chat. Redis for timers and pub/sub.
- Health endpoint reporting chain head, indexer lag, operator balance and Circle API errors.

## Frontend (Next.js, TypeScript, Tailwind)

Screens:
1. Landing. One-line pitch: "Every move is a transaction. The chain counts the votes." Big Play button, plus "Create private room".
2. Sign in with X (OAuth 2.0). Use the handle for display and the share card. Fall back to a guest name if X sign-in is unavailable.
3. Lobby. Seats fill live; bots fill after 20 seconds unless the room is private.
4. Game table. Council seats in a ring, the current page in the centre with its 5 transactions, your private balances in a side drawer, phase timer, chat, and Seal / Reject buttons. Your role shows once at the start and stays on a hold-to-peek control.
5. Live chain feed on the right edge: each move as it lands, for example "Ada sealed · finalised in 0.41 s · block 1,204,377", each linking to the explorer.
6. Finality moment. When a page passes 2/3 it locks with a stamp: "FINAL. Irreversible." When a forgery is caught: "Forgery caught". When a round stalls, the stall meter ticks.
7. End screen. Winner, every role revealed, the seed check result ("Dealt fairly: seed verified on chain"), and the line "This game was N transactions on Arc. Verify it." with a link.
8. Share card, generated server-side as an image: X handle, role, outcome, pages sealed, forgeries caught, transactions in this game, and the game link. One-click post to X.
9. Replay page /g/[gameId]. Rebuilds the full game purely from on-chain events and the published seed, step by step, for anyone.
10. About page. Explain the 2/3 rule, safety and liveness in plain words, and say honestly what is and is not trustless: the contract counts and checks everything, but in this version the server holds player wallets and salts, so it could read sealed votes during play. Arc Privacy Sector will remove that when it ships.

11. Endings. Three full-screen moments from docs/story.md: Finality (seals glow, confetti, "FINAL. Irreversible."), the Crack (the city splits in half, coral), the Freeze (frost and stopped clocks, cyan).
12. Daily Ledger. One shared seed per UTC day, everyone plays the same deal against bots, and the result shares as the emoji grid in docs/rules.md, with a daily leaderboard.

Design: follow docs/design.md exactly. It is a cartoon sticker-book system: pale sky background, every surface outlined in ink with a solid offset shadow (no blur), chunky rounded corners, Baloo 2 / Nunito / JetBrains Mono, springy motion, and expressive SVG Sealer characters with blinking eyes and moods. Do not use a default component library look, glassmorphism, gradients as decoration, or a dark generic dashboard. Mobile first at 375px, because most players arrive from X on a phone.

## Rules for this build

- No money stakes, no USDC prizes, no buying advantages. Points and cards only.
- No real people, companies or validators as characters. Invent every name.
- Testnet first; mainnet only by changing config. Same contract, same code.
- No private keys, API keys or entity secrets in client code or the repo. Provide .env.example.
- Every claim on screen about time or cost is measured, never hard-coded.

## Deliverables

Contract with tests, server, frontend, indexer, docker-compose for Postgres and Redis, a deploy script for the contract on ARC-TESTNET and ARC, .env.example, and a README covering setup, how to run a local game with bots only, how to deploy, and how the fairness check works.

Acceptance: a single visitor can sign in, play a full 7-seat game against bots on ARC-TESTNET, see every move in the live feed with measured finality, finish on the end screen with a passing seed check, open the replay page and watch the same game rebuilt from chain events, and download a share card.
