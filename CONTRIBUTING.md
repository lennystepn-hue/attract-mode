# Contributing

Thanks for looking. New palettes, new card types and better documentation are
all welcome, as are bug reports with a screenshot.

## Getting set up

Node 20 or newer. There is nothing to install.

```bash
GITHUB_TOKEN=$(gh auth token) npm run build
```

That fetches live data for whoever `profile.config.json` points at, writes
`assets/*.svg`, and saves a snapshot to `.cache/`. After the first run, iterate
without touching the network:

```bash
npm run build:cached
```

Open `preview.html` to see every card with its animations, and use the replay
button to restart them.

Preview a palette without editing the config:

```bash
npm run build:cached -- --theme phosphor
```

## The layout

```
profile.config.json   what a user edits; nothing else should need touching
scripts/
  config.mjs      loads and validates the config
  pixelfont.mjs   the 5x7 bitmap font, rendered to SVG paths
  sprites.mjs     agent, ghost and mascot bitmaps
  theme.mjs       palettes, language colours, seeded PRNG
  fx.mjs          CRT treatment, panels, meters, digit roll
  cards.mjs       the six compositions
  data.mjs        GraphQL call plus the public contribution graph
  readme.mjs      builds README.md from the same snapshot
  generate.mjs    entry point
```

## Checking your work

**The build log will not tell you that text has overflowed a panel.** Layout
bugs are invisible until you look at the output, and several have shipped that
way: a four-digit score running off the player card, description text colliding
with a project card's footer, two month labels landing on top of each other.

Rasterise and actually look:

```bash
npm i --no-save @resvg/resvg-js
```

```bash
node -e "const{Resvg}=require('@resvg/resvg-js'),fs=require('fs');for(const f of fs.readdirSync('assets'))if(f.endsWith('.svg'))fs.writeFileSync('/tmp/'+f+'.png',new Resvg(fs.readFileSync('assets/'+f,'utf8'),{fitTo:{mode:'width',value:900},background:'#14101f'}).render().asPng())"
```

resvg ignores CSS animation, so you get the resting frame — which is exactly the
state that has to contain correct data.

Render each file **separately**. Ids like `#marquee` and `#ttl` are
document-scoped, so merging several cards into one SVG to make a contact sheet
silently breaks every reference after the first.

## Rules that are not style preferences

These exist because breaking them produces bugs that are hard to see and easy to
ship.

1. **Never display a figure that depends on which token fetched it.** GraphQL
   reports commit, pull-request and issue counts relative to the caller. A
   personal token with `read:user` sees several hundred more than the workflow's
   repo-scoped one, so those numbers would flip on every scheduled run. Only
   calendar-derived and public-repository figures go on a card. `data.mjs` marks
   the token-dependent fields as deliberately unrendered.

2. **Keep randomness seeded.** Star positions and digit decoys come from
   `mulberry32`. A stray `Math.random()` makes every nightly run commit a diff of
   shuffled pixels.

3. **The resting state must be the truth.** Anything animated has to show the
   correct value with animations disabled. The digit roll puts the real digit
   first at offset zero for precisely this reason.

4. **Assume no fonts, no scripts, no network.** SVGs in a README load in *secure
   animated mode*: CSS `@keyframes`, SMIL, filters, `mix-blend-mode` and
   `<use href>` work; web fonts, `<a>` links inside the SVG, and JavaScript do
   not. [HOW-IT-WORKS.md](HOW-IT-WORKS.md) has the details and the traps.

## Adding a palette

Add a complete entry to `PALETTES` in `scripts/theme.mjs`. Every key in the
existing themes must be present, including `ramp` (five steps, dark to bright,
for the contribution map) and `marquee` (three stops for the hero gradient).

Two things to get right: `green` is reserved for live data, so it should read as
"positive" rather than decorative; and the background should be tinted toward
the accent rather than pure black, which never appears in nature and always
looks flat.

Then look at all four heroes side by side before opening the pull request.

## Commit messages

Explain the reasoning, not the diff. A message saying *why* the score digits are
ordered the way they are saves the next person from "fixing" it. Imperative
subject lines.
