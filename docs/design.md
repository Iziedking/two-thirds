# Design

Two-Thirds shares the Zerun design system: a bright sticker-book world, every
surface outlined in deep ink with a hard offset shadow, chunky rounded type,
springy motion, and characters that carry the personality. It must never look
like a generic dashboard or a default component library.

## Tokens

```css
:root {
  --sky: #EAEDFF;        /* page background, never white or cream */
  --sky-2: #DDE2FF;
  --cloud: #FFFFFF;      /* card fill */
  --cloud-2: #F6F7FF;

  --ink: #171449;        /* every outline, every shadow, all text */
  --ink-2: #4A477E;
  --ink-3: #8A88B8;

  --violet: #6C4CF1;     /* primary action, one per view */
  --violet-deep: #5436D6;
  --amber: #FFB13C;      /* seals, stamps, rewards */
  --mint: #1FD6A6;       /* final, live, success */
  --cyan: #36C5FF;       /* info, the Freeze */
  --coral: #FF6B5C;      /* forgery, the Crack, danger */

  --pop: 4px 4px 0 var(--ink);   /* solid shadow, no blur, ever */
  --pop-lg: 6px 6px 0 var(--ink);
  --pop-press: 1px 1px 0 var(--ink);

  --line: 3px;
  --r: 18px;
  --r-lg: 28px;
  --r-pill: 999px;
}
```

- Everything with substance has a `--line` ink outline and a `--pop` shadow.
- Corners always rounded. No soft shadows, no gradients as decoration, no glass.
- Ink is the only outline and shadow colour.

## Type

- Display: Baloo 2, 800. Titles, stamps, big numbers. May tilt -2 to -3 degrees
  on purpose, never everywhere.
- Body and UI: Nunito, 400/600/800.
- Mono: JetBrains Mono, only for transaction hashes, block numbers and addresses.

## Components

- **PopButton.** Rounded, outlined, `--pop`. Press squishes: translate 2px and
  drop to `--pop-press`. Primary violet, secondary amber, ghost cloud.
- **StickerCard.** Cloud fill, outline, `--r-lg`, `--pop`. Feature tiles may sit
  at ±1.5 degrees like a stuck-on sticker; data never tilts.
- **Sealer.** An SVG character with blinking eyes and moods: idle, thinking,
  suspicious, stamping, slashed. The player's seal floats beside it.
- **The Page.** A paper sticker with five transaction lines. Stamps land on it
  with a springy thunk and stack visibly toward the threshold.
- **Threshold meter.** Shows stamps so far against stamps needed, so one missing
  stamp is visible to everyone.
- **Chain feed.** A narrow rail of chips, one per move: who, what, finalised in
  N s, block number in mono, link to the explorer.
- **Chips.** Pills for FINAL (mint), FORGERY (coral), STALL (cyan), LIVE (mint).

## The three endings on screen

- Finality: every seal on the page glows amber, confetti, the page locks with a
  "FINAL. Irreversible." stamp.
- The Crack: the city illustration splits down the middle and the halves drift
  apart, coral.
- The Freeze: frost creeps in from the edges, clocks stop, cyan.

## Motion

Springy, short, purposeful: stamps, the threshold meter, endings, buttons.
Nothing moves just to move. Respect `prefers-reduced-motion`.

## Mobile first

Most players arrive from X on a phone. The table, the page and the stamp
buttons must work one-handed at 375px wide; the chain feed collapses into a
bottom sheet.
