---
name: zine-page
description: Generates a new editorial-zine HTML page for any serialized digital zine, matching the host repo's typography, masthead/hero/closing shell, color discipline, and per-page inline-override pattern. Use when adding a new Part/chapter/issue page, when the user asks to create/make/build an HTML page about a topic (including a request phrased as a plain or standalone HTML page), formatting pasted copy in the zine style, or scaffolding a serialized HTML zine in a fresh repo.
---

# Zine Page Builder

Generates a new editorial-zine HTML page for **any serialized digital zine** — matching the host repo's typography, masthead/hero/closing shell, color discipline, and per-page inline-override pattern. This is the **theme-agnostic** version zine-page skill.

## Prerequisites (one-time setup per repo)

Before using this skill, the host repo should provide:

1. **A shared stylesheet** at a known path (default: `css/global.css`) that defines design tokens (`--bg`, `--ink`, `--accent`, `--display`, `--body`, `--mono`, `--container-max`) and the canonical shell classes (`.masthead`, `.hero`, `.hero-grid`, `.part-tag`, `.hero-meta`, `section`, `.section-label`, `.lede`, `.bridge`, `.page-nav`, `.prev-pill`, `.next-pill`, `footer`, `.colophon`).
2. **A `zine.config.json`** (or equivalent) at the repo root declaring the series identity. See [Series config](#series-config) below.
3. **At least one finished page** to use as a visual reference. Without it, the skill will produce a structurally correct but stylistically generic result.

If any of these are missing, the skill handles it in [Step 0](#step-0-seed-or-generate-globalcss) rather than failing — either by seeding the fallback `default-global.css` from `assets/` or by generating a custom `global.css` from a mood board the user supplies.

## When to Use

Trigger this skill when the user:
- Asks for a new "Part", "chapter", "issue", or "section" page in a digital zine
- Pastes raw copy (headline, paragraphs, quotes, bullet evidence, code snippets) and asks for it formatted "like the others" / "in the zine style" / "matching the series"
- Says "add the next part", "make a Part N about X", "turn this into a zine page"
- Says "create me an HTML page about X", "make a page about X", "build an HTML page about X", "create me a plain / standalone HTML page", or any direct request to author a new page — this is the skill's core purpose. In this zine repo every page is a zine page, so even a "plain" or "standalone" request gets the zine shell. Run the Decision Gate and Step 2 survey to place it in the series; do not require the user to say the words "Part" or "zine".

Do **not** use this skill for:
- Edits to a single existing page (just edit directly)
- README or docs
- Requests where a mood board (reference images) is attached — hand off to moodboard-to-html (generic aesthetic) or moodboard-to-template (zine aesthetic)
- Adding new shared styles to the global stylesheet (do that directly)
- Single-page tabbed, accordion, or carousel layouts that hide inactive content via `hidden` / `display: none`. The skill's reading model is one document per Part. If a future archetype adds a stacked single-page layout (the `single-page-scrolling` archetype on the roadmap), it will use in-page anchors and stacked sections, not a tab widget.


## What it Produces

A single self-contained `<page-prefix>-N-<slug>.html` file at the configured location that:
- Links the configured shared stylesheet
- Loads the configured display + body + mono fonts from Google Fonts
- Uses the canonical shell: `.masthead` → hero band → body sections → bridge/closing band → `.page-nav` (prev/next pills) → `<footer>`
- Has an inline `<style>` block with page-specific overrides only (accent color, layout, scene)
- Uses italic `<em>` on accent words inside `h1` / `h2` if the series uses that convention
- Wires `prev`/`next` links to the surrounding pages

## Series Config

The skill reads `zine.config.json` (or accepts these as runtime inputs) to know the series identity:

```json
{
  "seriesName": "The HVE Issue",
  "year": "2026",
  "stylesheet": "css/global.css",
  "pagePrefix": "Part",
  "pageGlob": "Part-*.html",
  "fileLocation": ".",
  "fonts": {
    "display": "Fraunces",
    "body": "Fraunces",
    "mono": "JetBrains Mono",
    "extras": ["Caveat", "Permanent Marker"]
  },
  "footerLine": "Demo repo · FirstStreet Savings & Loan",
  "useItalicEmAccent": true,
  "archetypes": ["editorial", "noir-wall", "themed-scene", "visual-diagram"]
}
```

If `zine.config.json` is missing, ask the user once for each field, then offer to write the config file so future runs are zero-config.

## Procedure

### Decision gate: how many pages?

Before running any other step, resolve this question with the user. The skill produces serialized pages — one HTML file per Part, linked by prev/next pills (the canonical model). Phrases like *"3-part series"*, *"3 chapters"*, or *"3 sections"* are ambiguous. Ask:

> *"This skill produces serialized pages — one HTML file per Part, linked by prev/next pills (the canonical model). Did you mean:*
> *(a) Three separate Part files in a folder, each a full Part (recommended; what this skill is built for).*
> *(b) One bundled scrolling page with all three parts stacked, navigated by in-page anchors. Note: this is a future archetype; today the skill only emits (a).*
> *(c) Something else — describe what you want."*

If the user picks (a), proceed to Step 0. If the user picks (b) or (c), **stop** and surface the limitation rather than improvising a single-page tabbed, accordion, or carousel layout — that is explicitly out of scope per the "Do not use this skill for" list above.

### Step 0: Seed or generate `global.css`

**Subfolder placement precondition.** Before asking the Step 0 question, confirm the host repo path where `stylesheet` should land. If the series lives in a subfolder (for example `guidance/getting-started/`), the user's `zine.config.json` `stylesheet` field is relative to the repo root and the page files go in `fileLocation`. Do not assume `./css/global.css`.

Run this step **only when the configured stylesheet does not exist yet** (first-time setup in a new repo). If `css/global.css` already exists, skip to Step 1.

Ask the user one question:

> *"No `global.css` found. How do you want to theme this zine?*
> *(a) Use the bundled default (warm neutral, system fonts) — fastest, you can edit later.*
> *(b) Generate one from a mood board — paste 1–6 reference images and I'll derive the palette + fonts.*
> *(c) I'll write it myself — just scaffold the file with the token contract and let me fill it in."*

#### Path (a) — Bundled default

Copy `assets/default-global.css` from this skill folder to the configured stylesheet path (default `css/global.css`). The bundled default ships with:

- **Palette**: warm cream paper (`#F4EFE6`), deep ink (`#1A1614`), editorial navy accent (`#1A2E4A`), with a mustard `--highlight` (`#F5C842`) for emphasis blocks.
- **Fonts**: Fraunces (display and body), JetBrains Mono (code), loaded inline via `@import`, so no `<link>` tags needed in pages. Inter Tight remains a recommended sans alternative; swap it into `--body` to soften long-form reading. Headings `h1` and `h2` pin Fraunces' `opsz` axis to `144`, and their italic `<em>` accents also pull the `SOFT` axis to `100`, so any per-page heading override must preserve those `font-variation-settings`.
  - **Performance mode (optional)**: `@import` adds one extra serial round-trip to first paint because the browser's preload scanner cannot see CSS-embedded URLs. The `&display=swap` parameter keeps text visible during that window, so the page is never blank. If you need faster font arrival, delete the `@import` line from `default-global.css` and add this to your page template's `<head>` instead:

    ```html
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,400;0,9..144,500;0,9..144,700;0,9..144,900;1,9..144,400;1,9..144,700&family=JetBrains+Mono:wght@400;500;600;700&display=swap">
    ```

    The CSS will continue to consume the families through `var(--display)`, `var(--body)`, and `var(--mono)`.
- **Shell**: all canonical classes (`.masthead`, `.hero`, `.bridge`, `.page-nav`, `.prev-pill`, `.next-pill`, etc.) wired and ready.

Tell the user where it landed and that they can edit `:root` tokens any time to re-skin the whole series — or run Step 0 again with Path (b) to derive a custom palette from images.

#### Path (b) — Mood-board generation

The user supplies 1–6 images (book covers, magazine spreads, photographs, screenshots of sites or posters they admire). Inspect each image and derive:

| Token | How to derive |
|---|---|
| `--bg` | Most dominant warm/cool neutral across the boards (paper, wall, sky). |
| `--bg-card` | A slightly lighter or tinted variant of `--bg` (~5% lift). |
| `--ink` | The darkest non-accent color used for type — true black is rarely right; lean to deep brown, navy, or charcoal. |
| `--ink-soft` | A 60% mix of `--ink` toward `--bg`. |
| `--ink-faint` | A 35% mix of `--ink` toward `--bg`. |
| `--rule` | Same as `--ink` unless the boards show a distinct hairline color. |
| `--accent` | The single most-repeated saturated color across the boards. |
| `--accent-soft` | A 15% tint of `--accent` on `--bg`. |
| `--accent-bright` / `--accent-deep` | A lighter and darker sibling of `--accent` for hover/depth. |
| `--warn`, `--highlight`, `--good`, `--moss` | Pick from the boards if present; otherwise carry the bundled defaults. |
| `--display` | A serif (or strong display) family that matches the headline energy of the boards. Pull from Google Fonts. |
| `--body` | A clean humanist sans for body copy. Default to Inter Tight or Source Sans 3 if unsure. |
| `--mono` | A monospaced family with personality (JetBrains Mono, IBM Plex Mono, Fira Code). |

Process:

1. State what you observed in each image in one short line each (e.g., *"Image 1: editorial fashion spread, ivory paper, oxblood accent, Didone serif."*). This gives the user a chance to course-correct before you write code.
2. Propose the full token set as a small table. **Ask the user to confirm or adjust** before writing the file.
3. Once confirmed, take `assets/default-global.css` as the structural scaffold and replace only the `:root` block + the font-family declarations. Do **not** invent new shell classes — the contract listed in [Prerequisites](#prerequisites-one-time-setup-per-repo) is fixed so pages stay portable.
4. Update `zine.config.json` `fonts` field to match the new families.
5. Write the file to the configured stylesheet path.

##### Worked example (few-shot)

User drops in three images and says *"theme it like this."*

**Step 1 — Observe (one line per image):**

> *Image 1: Penguin Classics paperback cover — cream paper, deep oxblood spine band, thin black rules, condensed serif title set in small caps.*
> *Image 2: Aperture magazine spread — wide ivory margins, charcoal body type, a single muted teal pull-quote, photo bled to the gutter.*
> *Image 3: Letterpress poster — warm off-white stock, brick-red display type, hand-set wood-type feel, black hairline borders.*

**Step 2 — Propose tokens (table back to user):**

| Token | Value | Why |
|---|---|---|
| `--bg` | `#F1E9D9` | Average of all three paper stocks (cream/ivory/off-white). |
| `--bg-card` | `#F8F1E2` | ~5% lift of `--bg`. |
| `--ink` | `#1F1612` | Charcoal from image 2, slightly warmed to match the paper. Avoids true black. |
| `--ink-soft` | `#6B5C52` | 60% mix of `--ink` toward `--bg`. |
| `--ink-faint` | `#A99B8C` | 35% mix. |
| `--rule` | `#1F1612` | Same as `--ink` (hairline rules in images 1 and 3). |
| `--accent` | `#8B2E1F` | Oxblood/brick red — the dominant saturated color across images 1 and 3. |
| `--accent-soft` | `#E7D2CC` | 15% tint of `--accent` on `--bg`. |
| `--accent-bright` | `#B8503E` | Lighter sibling for hover. |
| `--accent-deep` | `#5E1E14` | Darker sibling for depth. |
| `--highlight` | `#5A8A87` | The teal pull-quote from image 2 — keep it as a secondary highlight. |
| `--display` | `'Playfair Display', Georgia, serif` | Matches the high-contrast serif energy of images 1 and 3. |
| `--body` | `'Source Sans 3', system-ui, sans-serif` | Clean humanist sans pairs with Playfair; reads like image 2's body type. |
| `--mono` | `'IBM Plex Mono', ui-monospace, monospace` | Slight typewriter warmth, fits the letterpress feel. |

> *"Approve, tweak any cell, or say 'try again' and I'll re-derive."*

**Step 3 — On approval**, write `css/global.css` with these tokens in `:root`, update `zine.config.json` `fonts` to `{display: "Playfair Display", body: "Source Sans 3", mono: "IBM Plex Mono"}`, and confirm in chat.

If the user says *"the accent is too dark, lift it"* — adjust only that row, don't re-derive everything.

#### Path (c) — Hand-rolled

Copy `assets/default-global.css` to the configured path but replace the `:root` block with a `TODO` comment listing every required token. Tell the user which tokens and shell classes are non-negotiable (per [Prerequisites](#prerequisites-one-time-setup-per-repo)), then hand back control.

### 1. Load series config

Read `zine.config.json`. If absent, run the one-question-at-a-time interview above and offer to persist the answers.

### 2. Survey the existing series

- Glob for files matching `pageGlob` at `fileLocation`
- The new file's number = highest existing N + 1, unless the user names a different slot
- `prev` = the file with N−1; `next` = the file with N+1 (or leave `next` as `#` if it doesn't exist yet)

If unclear which slot the user wants, ask once.

### 3. Choose the layout archetype

Pick based on the content the user pasted. If ambiguous, ask once with the options from `config.archetypes`. Default archetypes and when to use them:

| Archetype | Use when content is… |
|---|---|
| **editorial** (default) | Argument / essay / explainer with sections, ledes, pull-quotes, code blocks |
| **noir-wall** | "Evidence wall" / collage of artifacts (sticky notes, code snippets, tickets, scribbles) |
| **themed-scene** | Has an animated SVG centerpiece in the hero (orbit, twinkle, pulse, etc.) |
| **visual-diagram** | Dark page dominated by one big diagram block (strata, layered eras, system map) |

A host repo may define its own archetype names in `config.archetypes` and provide matching reference pages.

### 4. Collect page inputs

**Input detection** — first inspect what the user already pasted:

- If the message contains a clear headline + deck + at least one body section, treat it as a **structured paste**: parse directly, infer the archetype, and skip Q&A for fields you can confidently extract.
- If the message is just a topic / intent ("make a Part 12 about X"), switch to **guided Q&A**: ask one short question at a time, in order.
- If unclear, ask the user once which mode they want.

Decide these (ask only for what's missing):

- **Part number** and **slug** (kebab-case, used in filename + title)
- **Section label** (top-right of masthead, e.g. "Onboarding")
- **Kicker / part tag** (short uppercase line above the headline)
- **Headline** (mark which word(s) are italic `<em>` accent, if the series uses that convention)
- **Deck** (italic subhead under the headline)
- **Body content** (sections, quotes, code, bullets, etc.)
- **Closing line** (the editorial sign-off in the bridge band)
- **Accent color** (page-specific; defaults to whatever `--accent` is in the shared stylesheet)

### 5. Assemble the file

Use this baseline scaffold and adapt per archetype. **Always** link the configured stylesheet, **always** include the masthead + hero + bridge + page-nav + footer shell.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>{{TITLE}} · {{SERIES_NAME}}</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="{{GOOGLE_FONTS_URL_FROM_CONFIG}}" rel="stylesheet">
<link rel="stylesheet" href="{{STYLESHEET_FROM_CONFIG}}">
<style>
  :root {
    --accent: {{ACCENT_HEX}};
    --accent-soft: {{ACCENT_SOFT}};
  }
  /* Page-specific overrides only — never restate global tokens. */
</style>
</head>
<body>

<a class="skip-link" href="#main">Skip to content</a>

<div class="masthead">
  <div>{{SERIES_NAME}} · {{SECTION_LABEL}}</div>
  <div class="vol">No. {{PART_NUMBER_PADDED}}</div>
  <div>{{YEAR}}</div>
</div>

<main id="main">

<header class="hero">
  <div class="container">
    <div class="hero-grid">
      <div>
        <div class="part-tag">{{KICKER}}</div>
        <h1>{{HEADLINE_WITH_EM}}</h1>
      </div>
      <div class="hero-meta">
        <div class="label">In this part</div>
        <p>{{HERO_META_BLURB}}</p>
      </div>
    </div>
    <div class="hero-bottom">
      <p class="deck">{{DECK_WITH_EM}}</p>
      <p class="byline">{{BYLINE}}</p>
    </div>
  </div>
</header>

<!-- Repeat <section> per content unit -->
<section>
  <div class="container">
    <div class="section-label">
      <span class="num">01</span>
      <span class="lbl">{{SECTION_LABEL}}</span>
    </div>
    <h2>{{SECTION_HEADING_WITH_EM}}</h2>
    <p class="lede">{{SECTION_LEDE}}</p>
    <div class="prose">{{SECTION_BODY_HTML}}</div>
  </div>
</section>

<section class="bridge">
  <div class="container">
    <div class="bridge-tag">{{CLOSING_TAG}}</div>
    <h3>{{CLOSING_HEADLINE_WITH_EM}}</h3>
  </div>
</section>

</main>

<nav class="page-nav" aria-label="Series navigation">
  <a href="{{PREV_HREF}}" class="prev-pill">Prev · {{PREV_LABEL}}</a>
  <a href="{{NEXT_HREF}}" class="next-pill">Next · {{NEXT_LABEL}}</a>
</nav>

<footer>
  <div class="colophon">{{FOOTER_LINE}}</div>
</footer>

</body>
</html>
```

**Archetype variations** are layered on top of this scaffold via the inline `<style>` block:

- **editorial**: add `.deck`, `.prose`, `.pullquote`, `.codeblock` styling
- **noir-wall**: override `--bg` to dark, add `.scribble` / `.sticky` / `.stamp` / `.codeblock` with `.kw` / `.str` / `.cm` token classes, load Caveat + Permanent Marker
- **themed-scene**: add inline SVG centerpiece + CSS `@keyframes` animations, wrapped in `@media (prefers-reduced-motion: no-preference)`
- **visual-diagram**: override `--bg` to dark, dedicate one section to a large diagram block

**Code sections** (any archetype): use `.codeblock` for dark monospaced blocks. Inline tokens use `<span class="kw">`, `<span class="str">`, `<span class="cm">`, `<span class="red-hl">`, `<span class="yellow-hl">`.

**Animations** (themed-scene only): SVG-driven CSS `@keyframes` only. Never auto-playing JS animations, never anything that traps focus, never loops > ~15s. Always wrap in `@media (prefers-reduced-motion: no-preference)`.

Rules that apply to **every** archetype:

1. **Always link the configured stylesheet** — never inline shared tokens.
2. **Always include the Google Fonts preconnect** and the configured font families.
3. **Inline `<style>` is page-specific overrides only** — accent color, layout grids, scene CSS.
4. **Italic-em accent** on at least the headline if `config.useItalicEmAccent` is true. This is most series' visual signature.
5. **Masthead format**: left = `<seriesName> · <Section Label>`, center = `No. NN` (zero-padded), right = `<year>`.
6. **Always include `.page-nav` with prev/next pills** before the `<footer>`. Use `class="prev-pill"` / `class="next-pill"`.
7. **Footer**: one short line from `config.footerLine`, usually in the `.colophon` style.
8. **Body width**: respect `--container-max` via `.container` for editorial; noir-wall and visual-diagram are typically full-bleed.
9. **Never inline more than the per-page `:root` accent overrides plus page-only layout selectors.** If `css/global.css` does not exist on disk, **stop and run Step 0 first**. The skill must never produce a page whose inline `<style>` redeclares any of the global tokens (`--bg`, `--ink`, `--display`, `--body`, `--mono`, `--container-max`).

### 6. Write and confirm

Write the file to `{{fileLocation}}/{{pagePrefix}}-<N>-<slug>.html`. Then briefly tell the user:
- Filename created
- Archetype used
- Prev/next wired (or note if `next` is a placeholder)
- One offer: tweak accent color, swap archetype, or add another section

## Accessibility Bar

Target conformance: WCAG 2.2 Level AA. Every rule below cites at least one Success Criterion so authors can verify it. The bundled `default-global.css` ships browser-enforced defaults for focus visibility, the skip link, and reduced motion, so the rules in this section are the per-page semantic decisions that CSS cannot make for you.

### Document baseline

The page declares its language and structure so assistive tech can orient the reader.

* `<!DOCTYPE html>` is present and `<html lang="en">` is set (SC 3.1.1).
* `<title>` is unique to this page and matches the series number used in `prev`/`next` links (SC 2.4.2).
* `<meta name="viewport" content="width=device-width, initial-scale=1">` is present.
* Exactly one `<h1>` per page; subsequent headings do not skip levels, so the outline goes `h1` then `h2` then `h3` without gaps (SC 1.3.1).
* `<main id="main">` wraps the article body (everything between the masthead and the series prev/next nav) (SC 2.4.1).
* The first focusable element inside `<body>` is `<a class="skip-link" href="#main">Skip to content</a>` (SC 2.4.1).
* Each `<nav>` has an `aria-label` that distinguishes it from other navs on the page, for example `aria-label="Series navigation"` on the prev/next pills (SC 2.4.1, SC 4.1.2).
* Link text is descriptive of the destination on its own (SC 2.4.4).

### Color and contrast budget

The default palette contrast ratios against the cream background:

| Pair | Ratio | AA body (4.5:1) | AA large/non-text (3:1) |
|------|-------|-----------------|-------------------------|
| `#1A1614` on `#F4EFE6` (ink on cream) | ~15.7 : 1 | pass (AAA) | pass |
| `#1A2E4A` on `#F4EFE6` (navy on cream) | ~12.0 : 1 | pass (AAA) | pass |
| `#F5C842` on `#1A1614` (mustard on dark) | ~11.2 : 1 | pass (AAA) | pass |
| `#6F645B` faint on `#F4EFE6` (recommended) | ~5.0 : 1 | pass | pass |
| `#9A9087` faint on `#F4EFE6` (current default, do not use for body) | ~3.0 : 1 | FAIL | UI only |

Body text needs at least 4.5:1 against its background; large text (24 px or 18.66 px bold and up) and non-text UI elements need at least 3:1 (SC 1.4.3, SC 1.4.11). Never apply `--ink-faint` (or any color below 4.5:1) to body paragraphs. Dark archetypes that override `--bg` recompute every text color against the new background before shipping.

### Images, figures, SVG

Every `<img>` carries an `alt` attribute. Decorative images use `alt=""`. Inline SVG is one of two kinds (SC 1.1.1):

* Decorative SVG: add `aria-hidden="true"` and no title.
* Narrative SVG (hero diagrams, anything with meaning the reader needs): add `role="img"` and a child `<title>` element that captions the diagram in one sentence.

Wrap meaningful image-plus-caption pairs in `<figure>` with `<figcaption>` so the relationship is exposed (SC 1.3.1).

### Links and series navigation

Link text describes the destination on its own; never ship an arrow-only prev/next pill (SC 2.4.4). The prev/next pills set `rel="prev"` and `rel="next"` so series order is machine-readable. If a page has a table of contents, the current page is marked `aria-current="page"`. The `.page-nav` block sits in the same place on every page so reader orientation never shifts (SC 3.2.3, SC 3.2.4).

### Focus and keyboard

Never use `outline: none` without a `:focus-visible` replacement (SC 2.4.7, SC 2.4.11). The bundled `default-global.css` already provides:

```css
:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}
```

Every interactive element (links, `<details>`, any `<button>`) must be reachable and operable by keyboard alone (SC 2.1.1). Pointer-only gestures are forbidden (SC 2.5.8 raised the minimum target size and click area in WCAG 2.2).

A `<pre>` block that overflows horizontally is a scroll region, so it gets `tabindex="0" role="region" aria-label="Code sample: <lang>"` so keyboard users can scroll it. A `<pre>` that does not overflow does NOT get `tabindex`, because that would create a dead tab stop with no operable content.

### Motion

The bundled `default-global.css` ships a `prefers-reduced-motion: reduce` block that zeros out animations, transitions, and `scroll-behavior` globally. Do not override that block from a per-page `<style>`.

Allowed motion: subtle hover transitions on links, the global smooth scroll. Disallowed: loops longer than fifteen seconds, focus-trapping animations, auto-playing video (SC 2.3.3). Any motion that lasts five seconds or longer needs a visible pause, stop, or hide control near the trigger (SC 2.2.2).

### Typography

Body type is at least 16 px and line-height is at least 1.5 (SC 1.4.4, SC 1.4.12). Color is never the only signal: status, warnings, and "this is the bad one" callouts pair color with a text label, an icon, or a weight change (SC 1.4.1). The page renders at 200 % browser zoom without horizontal scrolling on the main article column (SC 1.4.4, SC 1.4.10).

### Pre-save checklist

Before writing the file, run through these yes/no items. Any "no" is a fix-now, not a defer.

1. `<!DOCTYPE html>` present, `<html lang="en">` set.
2. `<title>` is unique to this page and matches the series number in `prev`/`next` links.
3. `<meta name="viewport" content="width=device-width, initial-scale=1">` present.
4. Exactly one `<h1>` in the page; subsequent headings don't skip levels.
5. `<main id="main">` wraps the article body.
6. First focusable element is a working skip link pointing at `#main`.
7. Every `<nav>` has an `aria-label` distinguishing it from other navs.
8. Prev/next pills have descriptive text, never arrow-only.
9. Every `<img>` has `alt`; decorative images use `alt=""`.
10. Every inline SVG is either `aria-hidden="true"` (decorative) or has `role="img"` plus `<title>`.
11. No color-only signals (status, "this is the bad one", etc.); pair color with text, icon, or weight.
12. No body text uses `--ink-faint` or any color below 4.5:1 on the page background.
13. Dark archetypes recompute contrast against the overridden `--bg`; no raw `rgba()` below 4.5:1 for body text.
14. `<pre>` blocks with horizontal overflow have `tabindex="0" role="region" aria-label="Code sample: <lang>"`; non-scrolling `<pre>` does NOT have `tabindex`.
15. No `outline: none` without a matching `:focus-visible` replacement.
16. Page renders cleanly at 200 % browser zoom (no horizontal scroll on the article).

### Anti-patterns

* Arrow-only prev/next links.
* Headlines that skip levels (h1 then h3).
* Multiple `<h1>` per page.
* Missing `<main>` landmark.
* Decorative SVG without `aria-hidden`.
* Narrative SVG or hero diagram without `role="img"` plus `<title>`.
* Color used as the sole way to communicate (e.g. green = good, red = bad, with no label).
* Faint-ink applied to body paragraphs.
* Raw `rgba()` muted text on dark archetypes without verifying ratio.
* `outline: none` without a replacement focus indicator.
* Animations without a `prefers-reduced-motion` guard at the CSS level (the bundled `default-global.css` covers this globally, so don't override it per-page).
* `tabindex="0"` on a `<pre>` that isn't scrollable (creates a dead tab stop).

## Hero Layout Rule

The default `.hero-grid` is a two-column layout: a large display headline on the left, a `.hero-meta` "IN THIS PART" sidebar on the right. Three anti-patterns in per-page `<style>` blocks will silently break it.

* Do not restate `.hero-grid { grid-template-columns: ... }` without `minmax(0, 1fr)` on the headline track. A plain `1fr` lets the longest unbreakable word in the headline expand the track past the container, pushing `.hero-meta` off the right edge.
* Do not set `line-height` below `0.95` on `.hero h1` when the display font is italic and serif (Fraunces, Playfair, EB Garamond). Italic descenders on "y", "g", and "j" collide with baseline punctuation on the line below at any value tighter than that.
* Do not add `overflow: hidden` to `.hero` to mask a layout that overflows. That hides the bug instead of fixing it and can clip the focus ring on the skip-link target.

The bundled `default-global.css` already ships `grid-template-columns: minmax(0, 1fr) 280px`, `min-width: 0` on `.hero-grid > *`, `line-height: 0.95` on `.hero h1`, and `overflow-wrap: anywhere; hyphens: auto;` as escape hatches for headlines containing very long unbreakable words. Per-page overrides should preserve all of these.

## Editing Rules (don't drift the series)

- **Never** modify the shared stylesheet from inside this skill. If a token is missing, surface it to the user as a separate suggestion.
- **Never** import a font family not declared in `config.fonts`.
- **Never** use Tailwind, Bootstrap, or any CSS framework. Hand-rolled CSS only.
- Keep handwritten/marker accents (`.scribble`, `.sticky`, `.stamp`) **exclusive to noir-wall** pages unless the host series declares otherwise.
- The italic-em color trick on headlines is non-negotiable when `config.useItalicEmAccent` is true.

## Anti-patterns

- Producing a generic "modern landing page" with cards and gradients — this is an editorial zine, not a SaaS site.
- Putting body copy in `font-family: var(--display)` — display is for headlines and italic ledes only; body uses the configured body font, code uses the configured mono font.
- Skipping the prev/next pills (breaks the reading flow of the series).
- Inlining the full global palette in every page (defeats the shared stylesheet).
- Hard-coding the series name, year, or footer line — always read from `zine.config.json`.
- Wrapping `<code class="codeblock">` inside `<pre class="...">` where the `<pre>` selector is undefined, or using `<div class="codeblock">` without `white-space: pre-wrap`. The `.codeblock` class in `default-global.css` is designed for `<pre><code class="codeblock">...</code></pre>` and assumes the outer `<pre>` carries the default `white-space: pre` behavior. Per-page CSS must not redefine the class.

## Porting to a new repo

To use this skill in a fresh repo:

1. Copy the whole `.github/skills/zine-page/` folder (including `assets/default-global.css`) into the new repo.
2. Add a `zine.config.json` at the repo root — or let Step 0 prompt you for it on first run.
3. Trigger the skill. Step 0 will either seed the default CSS, generate one from a mood board, or scaffold a hand-rolled file.
4. Optionally hand-build the **first** page so the agent has a richer visual reference. From page 2 onward, the skill handles it.

The minimum portable bundle is just `SKILL.md` + `assets/default-global.css`. Everything else is generated or asked-for at runtime.

### Static-site-generator hosts

If the host repo is a static-site generator project, the emitted page may need to integrate with the generator's layout chain instead of standing alone. Detect the generator by these markers:

- **Jekyll** — `_config.yml` at the repo root.
- **MkDocs** — `mkdocs.yml` at the repo root.
- **Hugo** — `config.toml` or `hugo.toml` at the repo root.
- **Docusaurus** — `docusaurus.config.js` at the repo root.

When one of these is present, **ask the user** whether the page should integrate with the generator (for example, Jekyll integration means adding frontmatter such as `---\nlayout: none\npermalink: /...\n---` so Jekyll processes the page without wrapping it in the theme) or stand alone as raw HTML (accepting that the site theme will not wrap it and that relative `.md` links will not be rewritten). The default behavior is to ask, not to assume.
