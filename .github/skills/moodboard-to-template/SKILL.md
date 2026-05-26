---
name: moodboard-to-template
description: 'Convert mood board images into a paired zine style-guide HTML + matching template HTML + tokens.json for the zine aesthetic. WHEN: user attaches mood board images, asks for style guide from photos, design system from screenshots, mood board to HTML, extract design tokens from images, generate HTML template from references; USE FOR: vision-to-tokens extraction, single-file self-contained HTML output; DO NOT USE FOR: production design systems (use Material/Carbon/Polaris), Tailwind scaffolds, image editing, or sites without a mood board.'
---

# Moodboard → Template Skill

## 1. What this skill does

Reads attached mood board images, extracts a role-named design token set (color / typography / space / radius / shadow / motion), then emits three artifacts — a zine `style-guide.html`, a matching `template.html`, and the `tokens.json` they share. Both HTML files contain a byte-identical `:root` token block fenced by sentinels. Output follows the zine aesthetic.

## 2. Method

1. **Inspect each image.** Note dominant hues (sample large flat regions, not noise), value range (high-key / low-key / balanced), 3–5 mood adjectives, repeated motifs, era cues, photographic vs illustrative vs typographic.
2. **Resolve conflicts across images.**
   - **If exactly 1 image is attached:** skip conflict-resolution; record `meta.notes`: `"single-board input — no cross-image reconciliation performed"`. Lower defaulted tokens (`focus`, `border`, `shadow`) to `confidence ≤ 0.6`.
   - **If 2+ images:** cluster by similarity, keep the dominant cluster, name outliers in `meta.notes`. Never silently average warm + cool palettes.
3. **Emit the intermediate JSON token model** before any HTML. Role-named, DTCG-ish, with per-token `confidence` (0..1) and a top-level `a11y.checks[]`. Schema: `references/token-schema.json`.
4. **Validate WCAG 2.2 AA.** 4.5:1 body, 3:1 large/non-text. Auto-adjust offending tokens by ±10–15% L\*; record original + adjusted in `meta.notes` and surface a flagged badge in the style guide.
5. **Emit `style-guide.html`.** Single file, all CSS in `<head>`, sections per §5 below.
6. **Emit `template.html`.** Single file, masthead → hero → article → footer, mood-themed placeholder copy keyed to `meta.mood`.
7. **Self-check.** Verify the two files' `:root` blocks match byte-for-byte between `<!-- TOKENS:START -->` / `<!-- TOKENS:END -->`. List user follow-ups (uncertain tokens, dark-mode candidate, font licensing).

## 3. Token taxonomy

Six categories, semantic role names only — never numbered ramps like `--blue-500` at the consumption layer.

- **Color** — `surface`, `surface-alt`, `ink`, `ink-muted`, `accent`, `accent-ink`, `border`, `focus`, `code-bg`, `code-ink` (Dracula values).
- **Typography** — `font-display`, `font-body`, `font-mono`; size scale `text-xs`/`sm`/`base`/`lg`/`xl`/`2xl`/`3xl` (use `clamp()` for hero); `leading-tight`/`-normal`/`-loose`; `weight-regular`/`-bold`.
- **Spacing** — `space-1` through `space-8` (8 stops, 4px base, ~1.5× modular ratio: 4, 8, 12, 16, 24, 32, 48, 64).
- **Radius** — `radius-sm`, `radius-md`, `radius-lg`.
- **Shadow** — `shadow-sm`, `shadow-md`, `shadow-lg` (near-zero values when the board is flat).
- **Motion** — `motion-fast`/`-base`/`-slow`; `ease-standard`, `ease-emphasized`. Mandatory `@media (prefers-reduced-motion: reduce)` override.

Canonical `:root` block + reduced-motion + print overrides: [references/root-tokens.css](references/root-tokens.css).

## 4. Intermediate JSON token model

Loosely DTCG-compatible. Required top-level keys: `meta`, `color`, `typography`, `space`, `radius`, `shadow`, `motion`, `a11y`. Each color token carries `value`, `role`, `confidence`, and a `contrast` object. Each font token carries `value`, `confidence`, optional `system_safe` / `fallback_only` flags. `a11y.checks[]` lists every text/background pair the HTML actually uses, with computed ratio and `AA` / `AAA` / `FAIL` level.

Confidence calibration:

- `≥ 0.85` — directly sampled from a large region.
- `0.6 – 0.85` — inferred from mood / era cues.
- `< 0.6` — defaulted; **must** add `"note": "Defaulted; not derivable from board."`

Full JSON Schema (draft 2020-12): [references/token-schema.json](references/token-schema.json).

## 5. Output contract

Exactly **three** files, written next to each other in the chosen output directory:

1. **`style-guide.html`** — single self-contained file, all CSS in `<head>`, only Google Fonts external (Fraunces + JetBrains Mono with documented system fallbacks).
2. **`template.html`** — single self-contained file, same constraints.
3. **`tokens.json`** — the intermediate JSON token model that validates against `references/token-schema.json`.

**Self-check before returning:** (1) the two `:root` token blocks between `<!-- TOKENS:START -->` / `<!-- TOKENS:END -->` are byte-identical across both HTML files; (2) `tokens.json` byte-matches the `<details><summary>tokens.json</summary>` appendix embedded in the style guide; (3) every text/background pair the HTML uses appears in the WCAG `a11y.checks[]` table; (4) the mac-chrome triplet is exactly `#ff5f56` / `#ffbd2e` / `#27c93f`; (5) no inline `style=""` attributes contain raw values — only `style="--local: var(--global)"` overrides are allowed.

- **`style-guide.html`** — palette swatches (large swatch, role name, hex, contrast vs surface + ink with WCAG badge), typography specimens (display at xl/2xl/3xl, body paragraph at base + lg, mono code block), type scale ramp, spacing ruler, radius samples, shadow cards, motion demo (respects reduced-motion), components (primary / secondary / ghost button, input, card, callout, link), motif samples (mac window, tape stickers, hairlines), Dracula code-block sample, callouts, A11y report table, and a `<details><summary>tokens.json</summary><pre><code>` appendix that **must match `tokens.json` byte-for-byte**.
- **`template.html`** — masthead (issue / volume / date) → hero (kicker, dek, full-bleed band) → article (`p.lead` at `text-lg leading-loose`, `h2` at display, body paragraphs, `blockquote.pull-quote` with accent left border, `aside.sidebar` card) → footer (byline, credits, hairline). Placeholder copy is mood-themed, not lorem.
- **`tokens.json`** — standalone copy of the intermediate token model. Same bytes as the style-guide appendix.

## 6. Aesthetic charter

Full rules + verbatim CSS snippets: [references/zine-charter.md](references/zine-charter.md). The eight non-negotiables, summarized:

- Mac-chrome window wrapper for demo screens (3 traffic-light dots: `#ff5f56` / `#ffbd2e` / `#27c93f` — the only canonical triplet).
- Italic-as-accent on display type — `<em>` is italic *and* `color: var(--accent)`, with Fraunces `font-variation-settings: "opsz" 144, "SOFT" 100`.
- Cream paper + warm near-black ink default; Dracula duotone for code; no `prefers-color-scheme: dark` — dark sections are spotlights.
- Mono micro-labels: uppercase JetBrains Mono 10–13px, letter-spacing 0.08–0.25em, prefixed by `::before` accent hairline 28–36px × 1–2px.
- Editorial section numbering (`§ 01`, `STEP 01`) and bracketed zine asides.
- Fluid hero scale: `font-size: clamp(54px, 9.5vw, 200px); line-height: 0.82–0.95; letter-spacing: -0.025em to -0.05em;`.
- Self-contained single-file HTML, all CSS in `<head>`, Google Fonts only.
- Tokens fenced in `<!-- TOKENS:START -->` / `<!-- TOKENS:END -->` for byte-match sync.

## 7. Accessibility

WCAG 2.2 AA gates: **4.5:1** for body text, **3:1** for large text (≥18pt or ≥14pt bold) and non-text UI (borders, focus rings, meaningful icons). Compute and emit the ratio for every text/background pair the HTML uses. If a pair fails, auto-adjust the offending token by ±10–15% L\*, re-check, and surface both the original and adjusted values in the style-guide A11y table with a visible flag (red badge + ratio + adjustment note). Honor `prefers-reduced-motion` via the `@media` override in the canonical token block.

**Worked auto-adjustment example.** Initial sampled `border: #D6CBA9` measured 1.4:1 vs `surface` (FAIL for non-text UI). Lightness dropped ~12% L\* to `#8A7A55`, re-measured 3.5:1 (PASS for non-text UI). Both values surface in the style guide A11y table as `border: #D6CBA9 → #8A7A55 (FAIL 1.4:1 → PASS 3.5:1)`, and `meta.notes` records the adjustment.

## 8. Pitfalls (MUST NOT)

- **No hallucinated webfonts** the user can't load — require system-font fallbacks; provide a free Google Font alternative.
- **No accent-on-accent low-contrast text** — every accent must pair with an explicit `accent-ink` that passes 4.5:1.
- **No lorem-only output** — placeholder copy must be mood-themed and use real article structure (lead, h2, pull-quote, sidebar, byline).
- **No base64-embedded mood board images** in the output — describe sources in alt text and `meta.sources`.
- **No inline `style="…"` attributes** except the strict per-instance override form `style="--local: var(--global)"`. Never raw values (no `style="font-size:10px"`, no `style="color:#B8431C"`), never raw properties beyond a custom-property override.
- **No silent averaging of conflicting boards** — cluster, keep the dominant cluster, name the outlier in `meta.notes`.

## 9. References

- [references/root-tokens.css](references/root-tokens.css) — canonical `:root` block, reduced-motion + print overrides. Drop into both output files between the sync sentinels.
- [references/token-schema.json](references/token-schema.json) — JSON Schema (draft 2020-12) for the intermediate token model. Validate before emitting HTML.
- [references/zine-charter.md](references/zine-charter.md) — the eight non-negotiable aesthetic rules with verbatim CSS snippets.
