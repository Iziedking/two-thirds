# Rules

## Players and sides

5 to 10 players. Forkers = floor((n - 1) / 3), the largest number below a third.
Everyone else is a Sealer. Empty seats fill with clearly labelled bots.

| Players | Forkers | Stamps needed (more than 2/3) |
|---|---|---|
| 5 | 1 | 4 |
| 6 | 1 | 5 |
| 7 | 2 | 5 |
| 8 | 2 | 6 |
| 9 | 2 | 7 |
| 10 | 3 | 7 |

A page is final when `stamps * 3 > living * 2`. With 7 players and 2 Forkers a
page needs 5 stamps and there are exactly 5 honest players, so the honest side
moves only when all of them agree. One doubter freezes the city. That tension is
the game.

## Winning

- Sealers win on Finality: five honest pages stamped, or every Forker slashed.
- Forkers win on the Crack (a forged page stamped) or the Freeze (three stalled
  rounds).

## What each player knows

Each height the server deals a page of five transactions between citizens, such
as "Ada pays Bola 40 USDC". Each Sealer privately sees the true balances of two
or three citizens, so each can catch some forgeries and none can catch all.
Forkers see every balance and can lie about what they "know".

A forged page contains one transaction that spends more than the citizen holds.

## A round

A height can take several rounds. Each round has one proposer, living player
index `(height + round) mod living`.

| Phase | Time | What happens | On chain |
|---|---|---|---|
| Propose | 10 s | The proposer picks the page. An honest proposer always picks the valid page; a Forker proposer may pick a forgery. | Page hash |
| Debate | 45 s | Chat. Witnesses share balances, honestly or not. | No |
| Stamp | 15 s | Each living player commits stamp or reject. | Commitment |
| Reveal | 5 s | Commitments open. A missing reveal counts as not stamping. | Reveal |
| Close | instant | The contract counts. | Result |

Outcomes:
- More than two-thirds stamp a valid page: it is final, next height.
- More than two-thirds stamp a forged page: the Crack, Forkers win.
- Fewer stamp a forged page: "Forgery caught", same height, next proposer.
- Fewer stamp a valid page: a stall. Three stalls in a row: the Freeze.

## Slashing

After every second finalised height, each living player commits a target. The
plurality target is slashed; a tie slashes nobody. The slashed player's role is
revealed and checked on chain against the commitment posted at game start.
Slashed players watch. Thresholds recompute from the living count.

## Daily Ledger

One shared seed per day. Everyone plays the same deal against bots, and the
result shares as a grid with no spoilers:

```
Two-Thirds · Daily Ledger #14
🟩🟩🟥🟩⬜🟩🟩   Finality in 7
Caught 1 forgery · 61 txns on Arc
```

🟩 page stamped · 🟥 forgery caught · ⬜ stall · 💥 the Crack · 🧊 the Freeze.

## Progression (after launch)

- Titles from your record: Witness, Sealer, Keeper, Archivist.
- Feats: "Last Witness" (caught a forgery one stamp from final), "Clean Hands"
  (won as a Forker without proposing a forgery), and more.
- Seal designs unlocked by feats. Never sold.
- Epochs (seasons), each adding one role: Auditor (checks one citizen's true
  balance once), Light Client (sees less, stamp counts double), Double Agent
  (switches side at the midpoint).
- Arc House clubs as factions, a weekly Council Night, a club leaderboard.

## No money

No stakes, no USDC prizes, nothing to buy that changes play. Points, titles,
seals and cards only.
