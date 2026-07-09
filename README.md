# Use Case Portfolio Matrix

A single-file, dependency-free report page. Plots AI use cases by impact vs
complexity, ranks them by priority (impact ÷ complexity), and sequences them
into delivery waves.

Two tabs:

- **Dashboard** — the report: portfolio plot plus impact and complexity tables.
  Full-bleed, built for a projector or a screenshot.
- **Data** — a prompt to generate the JSON, and a box to paste it back in.

## Publish on GitHub Pages

1. Push `index.html`, `.nojekyll` and this README to a repo.
2. **Settings → Pages → Build and deployment → Source: Deploy from a branch.**
3. Branch `main`, folder `/ (root)`. Save.
4. Live at `https://<user>.github.io/<repo>/` in a minute or two.

`.nojekyll` stops Jekyll touching the output. No build step, no dependencies.

## Loading your own use cases

Open the **Data** tab, copy the prompt, paste it into Claude along with your
list of use cases, then paste the JSON it returns into the box and hit
**Load dashboard**. Everything stays in the browser — nothing is uploaded.

**Copy shareable link** packs the JSON into the URL fragment, so you can send a
populated dashboard to someone without deploying anything. Long lists make long
URLs; past roughly 20 use cases, edit `SAMPLE` in the source instead.

## Colour and pattern, one more time

Rank-chip colour follows **agentic fit** — the same green ramp used on the
chart bubbles — instead of quadrant identity. A pale mint chip and a deep
forest chip can sit in the same "Quick win" row; that's fit doing its job,
not a bug. Text colour on the chip flips automatically (dark ink for the
four paler fit steps, white for the deepest) to stay AA-compliant; see
`FITHEX` and the inline `color:` logic next to where `.rank` is built in
`drawTable()`.

Captions are now deliberately sparse, and all one colour (`var(--muted)`)
rather than colour-coded:

- **Impact table** — only Quick win and Big bet rows get a caption at all;
  Fill-in and Money pit rows are left blank. Quick win and Big bet no longer
  get different colours from each other — the caption is a label, not a
  status light.
- **Complexity table** — only two rows in the whole portfolio are captioned:
  whichever single use case has the *highest* complexity average ("Hardest
  overall") and whichever has the *lowest* ("Easiest overall") — just the
  tag, no driver breakdown attached. Everyone in between is left quiet.
  This is computed once in `compute()` — see `complexityTag` — by comparing
  every case's `cAvg`, not by re-deriving it per row. The per-case hardest/
  easiest driver (`c.hard`/`c.easy`) is still computed and sits in the
  tooltip data if you want to bring it back later.

The alarm colour still lives exactly where it did before: a `5` in the
complexity grid itself stays red, untouched by any of the above.

### Schema

```json
[
  {
    "name": "Retail Banking AI Chatbot",
    "short": "Retail Banking",
    "sub": "Online & mobile banking",
    "fit": 3,
    "reach": 72,
    "impact":     [5, 4, 4, 5, 5],
    "complexity": [5, 5, 2, 2, 4]
  }
]
```

`impact` is `[security, speed, quality, scale, roi]`.
`complexity` is `[data, security, human, docs, governance]`.
Both are five integers, 1–5. `fit` is 1–5. `sub` is optional.

`reach` (0–100) sets bubble area. **Omit it from every use case** and the
bubble-size encoding disappears cleanly, legend included. A size that means
nothing is worse than no size at all — so don't invent the number.

Quadrants, ranks, waves and the headline are all derived. Nothing is hardcoded.

## Brand colours

The true brand hexes are kept at the top of `:root` for reference:

```css
--cps-orange:#F26F21;
--cps-green :#2FA37C;
--cps-navy  :#062F3B;
```

But most of the page reads from a second set of *functional* tokens — same
hue family, deepened where the raw brand hex falls short of WCAG AA against
white:

```css
--win   :#B74B0B;  /* 5.2:1 — brand orange was 3.0:1 */
--ok    :#257F60;  /* 4.9:1 — brand green was 3.2:1 */
--fillq :#57646A;  /* 6.0:1 — the old fill-in grey was 2.5:1 */
--bet   :#062F3B;  /* 14.2:1 — navy was already fine, used as-is */
--alert :#A81E37;  /* 7.2:1 — already fine, used as-is */
```

Orange means *act now* — quick wins, the headline verb, the active tab,
the primary button. Navy carries the long-horizon big bets. Green is the
agentic-fit ramp, pale to deep, so higher fit reads as denser green — a
value scale rather than a flat status colour, so its individual steps
aren't held to the same AA bar as text. Crimson flags risk.

## Accessibility pass

Three changes, on request:

**Contrast.** `--muted` and `--faint` (captions, axis ticks, de-emphasised
table cells) were repainted from 3.4:1 / 2.0:1 up to 5.8:1 / 4.6:1 — both
now clear AA for normal text. Brand orange, green and the fill-in grey get
the same treatment above. Navy and crimson were already compliant and are
untouched.

**Line weight.** Lines now carry three tiers instead of one flat hairline:
*structural* (axis lines, the header rule, the Avg-column divider) uses
`--line-strong` at 3.3:1 with a heavier stroke; *semantic* (the dashed
quadrant thresholds) sits a step lighter, still clearly dashed; *decorative*
(row separators, background grid) stays faint, since it's genuinely
redundant with the numbers next to it.

**Pattern, not just colour, on the chart.** Quick win bubbles carry a dot
texture, big bets a diagonal-line texture — blended straight into the
fit-coloured fill with `mix-blend-mode:overlay`, so it reads as part of the
ball's surface rather than a decoration bolted on. Overlay blending keeps
the texture visible whether the ball underneath is the palest or deepest
step of the fit ramp. Fill-ins and money pits stay plain — texture is
reserved for the two quadrants worth calling out. The legend swatches use
the same dot/line treatment. `PATTERN_DEFS` near the top of `drawChart()`
defines both; `QUAD[...].pattern` maps quick win and big bet to their
`<pattern>` id (`null` for the other two).

## Thresholds

`T_IMPACT = 3.0` and `T_CPLX = 3.5` draw the quadrant lines. Move them and the
quick win / big bet split moves with them, along with the headline. They are a
judgement, not a law.
