---
name: moodboard-to-html
description: 'Generic, style-agnostic skill that turns a user-supplied mood board (images, font picks, color inspiration) into a reusable HTML template plus a matching style guide and tokens.json. WHEN: user attaches mood board images or describes a visual direction and asks for an HTML template, design system, or style guide from references; USE FOR: vision-to-tokens extraction, single-file self-contained HTML output, any aesthetic (not locked to a specific zine/brand); DO NOT USE FOR: the zine aesthetic specifically (use moodboard-to-template instead), production design systems (use Material/Carbon/Polaris), Tailwind scaffolds, or image editing.'
---

# Moodboard → HTML Template Skill (Reusable)

## 1. What this skill does

Reads a user-supplied **mood board** (attached images, font names, sampled colors, or free-text vibe description) and emits three artifacts that can seed any aesthetic:

1. `style-guide.html` — single self-contained file demonstrating every token.
2. `template.html` — single self-contained file showing the tokens applied to a real page (masthead → hero → content → footer).
3. `tokens.json` — the intermediate design-token model both HTML files consume.

The output is **style-agnostic**: the skill does not impose a specific aesthetic. It derives the look from whatever inputs the user provides.

## 2. Inputs the user can provide

Any combination of the following — the skill works with at least one:

- **Mood board images** — photos, screenshots, illustrations, posters, UI references.
- **Font inspiration** — explicit font names (Google Fonts preferred), descriptors ("serif with high contrast", "geometric sans"), or sample images of type.
- **Color inspiration** — hex/RGB values, named palettes, sampled swatches, or color adjectives ("sunbaked terracotta", "cool clinical blue").
- **Vibe / mood adjectives** — 3–7 words ("editorial, warm, scrappy", "brutalist, monochrome, dense").
- **Output directory** — where to write the three files. If not given, ask.
- **Optional reference style** — name an existing aesthetic to lean on (zine, swiss, brutalist, glassmorphic, neumorphic, terminal, magazine-editorial).

If the user provides **nothing visual** (only adjectives), say so and ask whether to proceed from adjectives alone or wait for images.

## 3. Method

1. **Inventory inputs.** List every image, font name, color, and adjective the user supplied. Record in `meta.sources[]`.
2. **Inspect each image.** Note dominant hues (sample large flat regions, not noise/JPEG artifacts), value range (high-key / low-key / balanced), 3–5 mood adjectives, repeated motifs, era cues, photographic vs illustrative vs typographic.
3. **Resolve conflicts.**
   - **1 image or adjectives-only:** skip cross-source reconciliation; record `meta.notes`: `"single-source input — no reconciliation performed"`. Lower defaulted tokens (`focus`, `border`, `shadow`) to `confidence ≤ 0.6`.
   - **2+ images:** cluster by similarity, keep the dominant cluster, name outliers in `meta.notes`. Never silently average warm + cool palettes.
4. **Emit `tokens.json` first.** Role-named, DTCG-ish, with per-token `confidence` (0..1) and a top-level `a11y.checks[]`. See §4.
5. **Validate WCAG 2.2 AA.** 4.5:1 body, 3:1 large text and non-text UI. Auto-adjust offending tokens by ±10–15% L\*; record original + adjusted in `meta.notes` and surface a flagged badge in the style guide.
6. **Emit `style-guide.html`.** Single file, all CSS in `<head>`, sections per §6.
7. **Emit `template.html`.** Single file, applies the same tokens to a realistic page. Placeholder copy is **mood-themed**, never lorem ipsum.
8. **Self-check.** Verify the two files' `:root` blocks match byte-for-byte between `<!-- TOKENS:START -->` / `<!-- TOKENS:END -->`. List uncertain tokens and follow-ups for the user.

## 4. Token taxonomy

Six categories, **semantic role names only** — never numbered ramps like `--blue-500` at the consumption layer.

- **Color** — `surface`, `surface-alt`, `ink`, `ink-muted`, `accent`, `accent-ink`, `border`, `focus`, `code-bg`, `code-ink`.
- **Typography** — `font-display`, `font-body`, `font-mono`; size scale `text-xs`/`sm`/`base`/`lg`/`xl`/`2xl`/`3xl` (use `clamp()` for hero); `leading-tight`/`-normal`/`-loose`; `weight-regular`/`-bold`.
- **Spacing** — `space-1` through `space-8` (4px base, ~1.5× modular ratio: 4, 8, 12, 16, 24, 32, 48, 64).
- **Radius** — `radius-sm`, `radius-md`, `radius-lg`.
- **Shadow** — `shadow-sm`, `shadow-md`, `shadow-lg` (near-zero values when the board is flat).
- **Motion** — `motion-fast`/`-base`/`-slow`; `ease-standard`, `ease-emphasized`. Mandatory `@media (prefers-reduced-motion: reduce)` override.

Token values are derived from the inputs — **the categories are fixed, but the values flex to the mood**.

## 5. Intermediate JSON token model

Loosely DTCG-compatible. Required top-level keys: `meta`, `color`, `typography`, `space`, `radius`, `shadow`, `motion`, `a11y`.

- Every color token carries `value`, `role`, `confidence`, and a `contrast` object (`vs_surface`, `vs_ink` ratios).
- Every font token carries `value`, `confidence`, `system_safe` (bool), and `fallback` (string).
- `a11y.checks[]` lists every text/background pair the HTML actually uses, with computed ratio and `AA` / `AAA` / `FAIL` level.

**Confidence calibration:**

- `≥ 0.85` — directly sampled or explicitly named by the user.
- `0.6 – 0.85` — inferred from mood / era cues.
- `< 0.6` — defaulted; **must** add `"note": "Defaulted; not derivable from inputs."`

## 6. Output contract

Exactly **three** files written to the user's chosen output directory:

1. **`style-guide.html`** — single self-contained file, all CSS in `<head>`. Only external resources allowed: Google Fonts. Sections required:
   - Palette swatches (large swatch, role name, hex, WCAG contrast vs surface + ink with badge).
   - Typography specimens (display at xl/2xl/3xl, body paragraph at base + lg, mono code block).
   - Type scale ramp.
   - Spacing ruler.
   - Radius samples.
   - Shadow cards.
   - Motion demo (respects `prefers-reduced-motion`).
   - Components: primary / secondary / ghost button, input, card, callout, link.
   - Code-block sample.
   - A11y report table (every pair the HTML uses).
   - `<details><summary>tokens.json</summary><pre><code>` appendix that **byte-matches `tokens.json`**.

2. **`template.html`** — single self-contained file, same constraints. Layout: masthead → hero → article (`p.lead`, `h2`, body paragraphs, `blockquote.pull-quote`, `aside.sidebar`) → footer. Placeholder copy is mood-themed.

3. **`tokens.json`** — standalone copy of the intermediate token model. Same bytes as the style-guide appendix.

**Self-check before returning:**

1. The two `:root` token blocks between `<!-- TOKENS:START -->` / `<!-- TOKENS:END -->` are **byte-identical** across both HTML files.
2. `tokens.json` byte-matches the `<details>` appendix in the style guide.
3. Every text/background pair the HTML uses appears in the `a11y.checks[]` table.
4. No inline `style=""` attributes contain raw values — only `style="--local: var(--global)"` overrides are allowed.
5. No base64-embedded mood board images in the output (describe sources in alt text + `meta.sources`).

## 7. Accessibility

WCAG 2.2 AA gates: **4.5:1** body text, **3:1** large text (≥18pt or ≥14pt bold) and non-text UI (borders, focus rings, meaningful icons). Compute and emit the ratio for every pair the HTML uses. If a pair fails, auto-adjust the offending token by ±10–15% L\*, re-check, and surface both original and adjusted values in the A11y table with a visible flag.

Honor `prefers-reduced-motion` via the `@media` override in the `:root` block.

## 8. Pitfalls (MUST NOT)

- **No hallucinated webfonts** the user cannot load — require system-font fallbacks with a free Google Font alternative.
- **No accent-on-accent low-contrast text** — every accent must pair with an explicit `accent-ink` that passes 4.5:1.
- **No lorem-only output** — placeholder copy must be mood-themed and use real article structure.
- **No base64-embedded mood board images** in the output.
- **No inline `style="…"` with raw values** — only `style="--local: var(--global)"`.
- **No silent averaging of conflicting boards** — cluster, keep the dominant cluster, name the outlier.
- **No imposed aesthetic** — this skill does not assume zine, swiss, brutalist, or any other style unless the user names one.

## 9. Differences from `moodboard-to-template`

- **This skill is aesthetic-neutral.** No mac-chrome window, no italic-as-accent rule, no Dracula code colors, no cream-paper default, no editorial section numbering — those are zine specifics.
- **Use this skill** when the user wants a fresh aesthetic derived purely from their inputs.
- **Use `moodboard-to-template`** when the user explicitly wants zine look.

## 10. Example prompts

- "Here are 3 mood board images and I want to use Fraunces + Inter. Make me a template."
- "My palette is `#1a1a2e`, `#16213e`, `#e94560`, `#f5f5f5`. Vibe: brutalist, dense, terminal. Generate the three files in `./output/`."
- "Build a style guide and HTML template from these screenshots. Use system fonts only."
