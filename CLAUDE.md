# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## Project

Single-file static website for **Nash Bakery** (@nashbakery__), a real bakery in
Damascus, Syria. Everything — HTML, CSS and JS — lives in `index.html`. No build
step, no package manager, no server: open the file in a browser to preview.

The file began life as a coffee-shop template (`q7-coffee-cinematic-hero.html`)
and was rebuilt for Nash. If you find anything still talking about espresso,
pour-over, Malki or "Q7", it is a leftover and should go.

## Business facts

Copy must stay consistent with these. They come from the bakery's Instagram, and
**nothing beyond this list is known** — do not invent details to fill a layout.

- Name **Nash Bakery**, Damascus, **est. 2018**.
- Hours **10:00–00:00, every day**. Phone **0987 514 666** (`tel:+963987514666`).
- Map pin `33.515793, 36.283978`. **No street address has ever been published** —
  link the coordinates, never write out a street.
- Products: cakes, cookies, cinnamon rolls, bread/buns/baguettes.
- Delivery via **BeeOrder** and **Movo**.
- **No prices are published.** Don't invent any. The menu points to the counter
  and the delivery apps instead, and that is deliberate.

## Brand direction

Warm editorial, derived from the studio reference the client chose (`example.jpg`
— an Anna Bradshaw Content Co. page by Cember Studio). The palette below was
sampled from that image rather than guessed, so keep it exact.

| Token | Value | Role |
|---|---|---|
| `--paper` | `#FAF6F3` | warm off-white ground |
| `--paper-2` | `#F2EBE2` | deeper cream, alternating bands |
| `--ink` | `#221E1B` | warm near-black, body text |
| `--green` | `#23524A` | deep teal-green, full-bleed panels |
| `--green-deep` | `#1A3E38` | footer |
| `--gold` | `#B8811A` | goldenrod — display type **on cream** (see Contrast) |
| `--butter` | `#F3C95E` | light gold — type **on green** |
| `--wine` | `#9E2B3A` | wordmark, stamp, CTA banner |

Two rules hold the palette together, and breaking either makes the page look like
a different site:

1. **Gold is only ever type. Green is only ever ground.** Never swap them.
2. Gold shifts value with its background — `--gold` on cream, `--butter` on green.
   This is what keeps the low-contrast reference look legible; `--butter` on cream
   fails contrast badly, so don't reach for it there.

**Type** — three families, all Google-hosted and actually loading:

- `--display` **Fraunces** — headings and product names. Warm, high-contrast,
  slightly wonky. Chosen over Playfair Display deliberately (see below).
- `--text` **Newsreader** — body copy and UI. Quiet reading serif.
- `--hand` **Caveat** — the handwritten margin notes. Used sparingly on
  purpose; scattering it further kills it.

Corners are essentially square; only `.pinned` photos take a 2px radius.

### Why the palette looks the way it does

`.agents/skills/frontend-design` flags "warm cream background + high-contrast
serif + terracotta accent (#D97757)" as the single commonest tell of an
AI-generated page. The client's reference pins cream and serif, so those stay —
but the accent axis was moved deliberately to goldenrod / teal-green / wine.
**Do not introduce terracotta, clay or rust-orange**, and don't drift the gold
toward orange. If a new section needs an accent, use one that is already here.

## Architecture

Sections in document order, each a `<section>` directly under `.page-frame`:

`hero` → `story` → `menu` → `visit` → `gallery` → `faq` → `cta-banner` → `footer`

Things worth knowing before editing:

- **The pinned photo** (`.pinned`) is the site's one repeated gesture: white
  border, faint shadow, slight rotation. It is used in the hero collage, the
  story panel, the menu's featured row and the map. New imagery should use it
  rather than inventing a second treatment.
- **The hero is full-bleed cinematic media** with the line at the bottom. The
  `<img>` is a **stand-in for real Nash footage** — swapping in
  `<video autoplay muted loop playsinline poster="…">` needs no CSS change,
  because `.hero-media > *` styles either. A video must be muted to autoplay at
  all, and wants a poster and a short loop: a heavy hero video is punishing on
  a Damascus mobile connection.

  Two things hold it together:

  1. **The navbar is `position:fixed`, not `sticky`.** Sticky keeps its place in
     the flow, so the hero started *below* it and a cream band cut across the
     top of the footage. Fixed also means anchor targets need
     `section[id]{scroll-margin-top}`, which is set to 86px — confirmed by
     jumping to `#story` directly rather than trusting a screenshot tool's own
     scroll-into-view, which ignores `scroll-margin-top` and will show a false
     overlap that never happens in real navigation.
  2. **The scrim is a four-stop gradient**, not a flat tint — dark at the top
     for the navbar, dark at the bottom for the line, clear through the middle
     so the photograph survives. White type over an unscrimmed photo is a coin
     toss.

  The slow push-in (`heroPush`) is what makes a still read as cinematic. It is
  removed outright under reduced motion; a continuously scaling full-screen
  image is a vestibular trigger.
- **The story panel is a two-photo collage**, not one: a large documentary
  shot of the counter with a smaller detail photo (hands kneading dough)
  pinned over its corner, plus a facts strip (Est. 2018 / Baked fresh every
  morning / Damascus) restating what the paragraph already says as a
  scannable line. This carries more weight than the original single-photo
  version because the hero now hands off directly to it — there used to be a
  steam scene between them, cut for feeling like decoration rather than
  content, and the story panel absorbed that section's job.

  Both photos use `.pinned` rather than a new treatment, per the rule above.
  Both also need `.story-shot, .story-inset{overflow:hidden;}` so the
  `data-rv="uncover"` reveal's image slide doesn't spill past the white
  border — `.pinned` itself does not clip.
- **Green panels** carry `.on-green`, which switches the focus ring and `.note`
  colour to `--butter`.
- **The menu is the showpiece.** It is a photo grid, not a list: every item
  carries its own photograph, because this is where people decide what to buy
  and it deserves the largest images on the page. One card per run carries
  `.is-lead` and spans two columns with a wider crop, which is what keeps it an
  editorial grid rather than a uniform tile field.

  The filter contract: `.mf-btn[data-show]` toggles `.is-on` and sets
  `data-view` on `.menu-board`; CSS then hides any `.mi` whose `data-cat`
  doesn't match. Categories are `all` / `cakes` / `small` / `bread`. Adding one
  means touching the buttons, the cards' `data-cat`, and the three-selector
  hide rule.
- **Visit open/closed** — `VISIT_HOURS` in the last script block, in minutes
  after midnight, evaluated in `Asia/Damascus`. Closing at midnight is `24 * 60`;
  the formatter mods by 1440 so it prints `00:00` rather than `24:00`.
- **Gallery strips** duplicate their `.strip-set` four times and animate to
  `translateX(-25%)`. Change the number of copies and you must change the
  percentage to match, or the loop will jump. The strip images deliberately
  carry **no `loading="lazy"`**: the duplicated sets sit outside the viewport
  horizontally, so lazy loading never fires for them and they scroll in blank.
  They are the same eight URLs four times over, so the copies come from cache.

## Motion

**The hero load sequence.** The line rises out of a mask so it reads as being
revealed by the footage rather than dropped on top of it, then the strapline
and the scroll cue follow.

**Scroll reveals**, which the client asked for directly. A previous version had
none, on the grounds that fade-up-on-every-section is a generated-page tell.
The compromise is that the reveal is chosen per content type rather than
applied uniformly:

- `data-rv="rise"` — headings rise out of a mask.
- `data-rv="uncover"` — photographs slide up *inside* a fixed frame, so it
  reads as a reveal rather than an arrival. The frame needs `overflow:hidden`.
- `data-rv="fade"` — supporting text only.

They run through `ScrollTrigger.batch`, so a screenful of menu cards animates
as one group instead of twenty independent triggers. Initial states are set in
**JS, never CSS**, so nothing is invisible if the script fails to run.

Reduced motion is tiered, not switched off (see `.agents/skills/accessible-animation`):

- **Removed outright:** Lenis smooth scroll, the gallery marquee, and the hero
  push-in — a continuously scaling full-screen image is a vestibular trigger.
- **Softened:** the hero sequence and every scroll reveal collapse to a short
  cross-fade with no displacement — the entrance survives, the movement does not.
- **Kept:** colour and focus transitions, the FAQ accordion.

## Skills

Project skills live in `.agents/skills/` (not the usual `.claude/skills/`), so
they are not auto-listed — read them as files. `frontend-design` and
`accessible-animation` shaped the decisions above. The GSAP family is relevant
if you touch the hero timeline; GSAP + ScrollTrigger + Lenis are loaded from CDN.

## Previewing

There is no build step, but **open the page over HTTP, not by double-clicking it**:

```
python -m http.server 8787     # then http://localhost:8787/index.html
```

The Google Maps embed refuses to render on a `file://` origin — it loads, it
just paints nothing, so the map looks like a blank white card. Everything else
works from the filesystem; only the map needs the server.

For screenshots, Playwright drives the installed Chrome
(`p.chromium.launch(channel="chrome")` — the bundled browser is not downloaded).
Scroll the whole page first or the lazy images are still blank when you capture.

### Contrast

The reference's gold-on-cream is beautiful and fails WCAG, so the gold here is
deepened to `#B8811A`, which clears 3:1 as large text. Two consequences worth
remembering:

- `--gold` is only safe at **24px and above**. That is why `.menu-group h3`
  has a 24px floor. Don't use gold for body-sized text on cream.
- `--muted` is at `#736A61` (4.9:1). Lightening it to look more elegant will
  push the menu descriptions below 4.5:1.

There is a checker in the session scratchpad; the quick version is that
`--ink-soft`, `--wine`, `--butter`-on-green and `--paper`-on-green all pass
comfortably, and only those two tokens are tight.

## Local assets

`images/` holds the only files that are not inline or hot-linked:

- `favicon.svg` / `apple-touch-icon.png` — a butterfly silhouette, left over
  from a scene that was cut. It works as an abstract mark, but nothing else on
  the page is a butterfly any more, so it is a loose end worth a decision.
- `cheesecake.webp` — the answer to the guessing, **cut out of its backdrop** so
  it floats on the green. This one is a *light subject on a near-black ground*,
  so it was cut by **luminance threshold** (120) and largest-component, not by
  the edge flood fill used elsewhere — a flood kept a shadow that was connected
  to the cake. Below ~120 the plate comes too; above ~145 the crust erodes.
- `og.jpg` — **stale**: it is a screenshot of a scene that no longer exists and
  needs regenerating. All were referenced in `<head>` long before they existed.

## Placeholders still to replace

Every other photograph is a stock Unsplash URL, marked with a comment in the
HTML. They are stand-ins for real Nash photography. When swapping them in, keep
the warm/dark tones — the pastel and brightly-lit stock shots that were
considered fight the cream-and-green palette badly.

The emblem is currently used in the rose scene and the favicon. It would also
sit naturally in the wine stamp on the story photo and in the footer, which is
what would make it read as Nash's mark rather than a one-off illustration.
