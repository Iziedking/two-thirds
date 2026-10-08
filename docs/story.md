# Story

## Arc City

Arc City floats above the clouds and runs on one book: the Ledger. Every promise
made in the city is written on a page, and once a page is stamped nobody can
ever tear it out.

Pages are stamped by the Sealers. Each Sealer carries a personal seal, and a page
becomes final only when more than two-thirds of the living Sealers stamp it.

Hidden among the Sealers are the Forkers. A Forker wants exactly what the name
says: to split the city in two. One forged page is enough, a payment that spends
the same money twice. If it is stamped, the Ledger disagrees with itself and the
city cracks down the middle.

## The three endings

Every game ends in one of three ways, and each one is a real outcome of BFT
consensus, the kind Arc's Malachite engine runs.

| Ending | What happened | What the city does | Consensus term |
|---|---|---|---|
| Finality | Five honest pages stamped, or every Forker slashed | Every seal glows, fireworks, the last page locks | Progress |
| The Crack | A forged page was stamped | The city splits in half, buildings drift apart | Safety failure |
| The Freeze | Three rounds in a row ended with nothing stamped | Every clock stops, the city frosts over | Liveness failure |

Players learn safety and liveness by losing to them, not by reading about them.

## Why two-thirds

The number is not decoration. It is the exact threshold Arc's validators use,
and it is why up to a third of the Council can be lying and the city still
stands. The game is balanced by that math, not by a designer's guess. See
[rules.md](rules.md) for the table.

## The cast

- **Sealers.** Small, expressive sticker characters in the same family as
  Zerun's agents: outlined, bouncy, with eyes that blink. Each player picks a
  Sealer and designs a seal; the seal appears on every page they stamp.
- **Forkers.** Look exactly like Sealers. That is the point.
- **Citizens.** The people in the transactions on each page: Ada, Bola, Chidi,
  Dana, Emeka, Femi, Gita, Hugo and more. Invented names only.

No real people, companies or validators appear anywhere in the game.

## Tone

Playful on the surface, sharp underneath. Accusations are funny, the Crack is
dramatic, the Freeze is eerie, Finality is a party. Copy is short, warm and
direct. Nothing reads like a whitepaper.
