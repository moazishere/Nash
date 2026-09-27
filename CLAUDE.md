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

- **The navbar** has the logo on the left and the links on the right (it used
  to centre a text wordmark; the client moved it). The logo is
  `images/logo-wordmark.png`, the NASH lettering only — the full lockup's
  "EST. 2018 / ARTISANAL BAKERY" lines shrink to ~4px at bar height. It is
  dark ink, and CSS turns it white (`filter:brightness(0) invert(1)`) while
  the bar floats over the hero. **At 760px and below** the links fold into a
  full-screen cream panel behind a two-line toggle, with a green "Call" button
  at the bottom. While open it locks the page (`overflow:hidden` plus
  `window.nashLenis.stop()` — Lenis is exposed on `window` for this), closes
  on Escape or on any link, and forces the bar to its cream state via
  `.nav-open`; that is why the transparent-bar rules read
  `:not(.scrolled):not(.nav-open)`.
- **The pinned photo** (`.pinned`) is the site's one repeated gesture: white
  border, faint shadow, slight rotation. It is used in the hero collage, the
  story panel, the menu's house favourites and the map. New imagery should use it
  rather than inventing a second treatment.
- **The hero plays real Nash footage**: `video.mp4` in the project root, a
  **vertical phone clip (360 x 640, ~35 s, 3.9 MB)** with
  `images/hero-poster.jpg` (its first frame) as the poster. It is `muted`
  with no controls, so it can never make a sound — the client asked for no
  sound, and autoplay only works muted anyway. The file still carries an
  audio track; stripping it (no ffmpeg on this machine) would shave a little
  weight off a Damascus mobile connection.

  Because the clip is vertical, the layout splits on `min-aspect-ratio: 1/1`:
  on portrait screens it fills the hero full-bleed; on landscape screens
  filling it would mean a ~4x upscale with the people cropped out, so it
  becomes a tilted white-bordered card on the right (the `.pinned` look) over
  a blurred copy of the poster, and the line moves to the left. The push-in
  runs on the video when it is full-bleed and on the blurred backdrop when it
  is a card. Under reduced motion the video is paused on its poster, in its
  own script block so it works even if GSAP fails to load. If the footage is
  ever swapped for a landscape clip, the card mode is what to remove.

  Two things hold it together:

  1. **The navbar is `position:fixed`, not `sticky`.** Sticky keeps its place in
     the flow, so the hero started *below* it and a cream band cut across the
     top of the footage. Fixed also means anchor targets need
     `section[id]{scroll-margin-top}`, which is set to 86px — confirmed by
     jumping to `#story` directly rather than trusting a screenshot tool's own
     scroll-into-view, which ignores `scroll-margin-top` and will show a false
     overlap that never happens in real navigation. **Lenis ignores
     `scroll-margin-top`** for clicked links, so the same 86px is also passed
     as `anchors:{offset:-86}` in the Lenis setup — change one, change both.
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
  carries its own photograph, because this is where people decide what to
  buy. It has two parts:

  1. **House favourites** (`.favs`, "What people ask for") — a `--paper-2`
     band of three `.pinned` photos, the largest on the menu so they lead.
     Only items the copy already singles out: pistachio cake (house
     favourite), cinnamon rolls, date maamoul (seasonal). They also stay in
     the grid below so every category is complete. On phones the row becomes
     a horizontal scroll-snap strip.
  2. **"What we bake"** (`.menu-main`) — **one grid, one category at a time.**
     A sticky side column holds the heading, the category tabs (with item
     counts filled in by JS) and `.menu-foot` (prices / whole cakes /
     delivery). Beside it, `.menu-grid` is **2 columns**: a category opens as
     a 2 x 2 and `.menu-more` ("Show all 6 cakes" / "Show fewer") reveals the
     rest; it hides itself when a category has 4 or fewer.

  The client's calls, in order, so they are not undone: the photos were too
  big (keep the grid beside the side column — a full-width 2-column grid
  brings the huge cards back, which is why the stack only happens at 640px);
  no "Everything" tab (the menu opens on Cakes); no separate section per
  category (an earlier version gave each a full-bleed green band — rejected).

  **Drinks replaced Bread** as the third category at the client's request.
  Drinks are **not** on the published product list above, so the five drink
  cards (coffee, latte, tea, hot chocolate, iced tea) are placeholders,
  marked with a comment in the HTML — swap in what Nash really serves. Bread
  is still a real product and still named in the story, meta and FAQ copy.

  The contract: each `.mi` carries `data-cat` (`cakes` / `small` / `drinks`),
  each `.mf-btn` a matching `data-show`, and the menu script sets `hidden` on
  every item outside the current category or past the first four. Visibility
  lives in JS, never the HTML, so without JS the whole menu shows. Footer
  links select a category through `a[data-menu]`. Adding a category means a
  new button, its items' `data-cat`, and a noun in `NOUN` for the button
  label. The grid's cards are revealed by the menu script, not by
  `data-rv="uncover"`: ScrollTrigger measures hidden items at zero height.
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
**JS, never CSS**, so nothing is invisible if the script fails to run. The
uncover tween ends with `clearProps:'transform'`: without it GSAP leaves an
inline `translate(0px, 0px)` on every photo, which silently beats the CSS
hover zoom.

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

- `logo.jpg` — **the client's logo, untouched**: black lettering on an olive
  square. Olive is not in the palette, so the site never shows the square;
  the ink was cut out by luminance (ground ~129, ink ~20, alpha ramped between
  with the bottom 10% dropped as JPEG noise) into two transparent PNGs:
  `logo-wordmark.png` (navbar) and `logo-full.png` (footer, in cream). Re-cut
  from `logo.jpg` if the logo changes; don't edit the PNGs by hand.
- `favicon.svg` / `apple-touch-icon.png` — a butterfly silhouette, left over
  from a scene that was cut. Now that the real logo is on the page, these
  should become the NASH wordmark (or an "N" from it) — still to do.
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
