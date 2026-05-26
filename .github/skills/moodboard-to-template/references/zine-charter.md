<!-- markdownlint-disable-file -->
# Amie Dansby Basics Distilled Style Charter

The eight non-negotiable rules. Generated HTML must satisfy every one.

- **Mac-chrome window wrapper for demo screens.** Three traffic-light dots + centered mono filename. Canonical triplet (only this set):

  ```css
  .window .title-bar .dot:nth-child(1) { background: #ff5f56; }
  .window .title-bar .dot:nth-child(2) { background: #ffbd2e; }
  .window .title-bar .dot:nth-child(3) { background: #27c93f; }
  ```

- **Italic-as-accent on display type.** Every `<em>` inside a display heading is italic *and* tinted with the page accent. Use Fraunces variable axes:

  ```css
  .hero h1 em {
    font-style: italic;
    color: var(--accent);
    font-variation-settings: "opsz" 144, "SOFT" 100;
  }
  ```

- **Cream paper + warm near-black ink + Dracula duotone for code.** No `prefers-color-scheme: dark` media query — dark sections are spotlights, not modes. Code blocks always use the Dracula palette via `--code-bg: #282a36` and `--code-ink: #f8f8f2`, referenced once through the canonical token block.

- **Mono micro-labels with leading hairline.** Uppercase JetBrains Mono 10–13px, letter-spacing 0.08–0.25em, prefixed by a `::before` accent hairline 28–36px × 1–2px:

  ```css
  .label, .kicker, .meta { font-family: var(--font-mono); text-transform: uppercase;
    font-size: 11px; letter-spacing: 0.18em; color: var(--accent); }
  .label::before { content:''; display:inline-block; width:32px; height:1.5px;
    background: var(--accent); vertical-align: middle; margin-right: 10px; }
  ```

- **Editorial section numbering and zine asides.** Number sections with `§ 01`, `STEP 01`, etc. Use bracketed/parenthetical asides like `[ Built from references — not copied from them ]`.

- **Fluid hero scale.** Headlines use `clamp()`, never fixed pixels:

  ```css
  .hero h1 {
    font-size: clamp(54px, 9.5vw, 200px);
    line-height: 0.82–0.95;
    letter-spacing: -0.025em to -0.05em;
  }
  ```

- **Self-contained single-file HTML.** All CSS in `<head>`. Only external request allowed: Google Fonts (Fraunces + JetBrains Mono) with documented system fallbacks. No build step.

- **Tokens fenced in sync sentinels.** Wrap `:root { … }` between `<!-- TOKENS:START -->` and `<!-- TOKENS:END -->` so style guide and template stay in sync and the user can copy-paste the block as one unit.
