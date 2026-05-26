---
title: SlideCraft HTML
description: VS Code Copilot skills that turn a mood board (or a sentence) into a styled HTML page.
---

[![Deploy instruction docs to GitHub Pages](https://github.com/AmieDD/slidecraft-html/actions/workflows/deploy-pages.yml/badge.svg?branch=main)](https://github.com/AmieDD/slidecraft-html/actions/workflows/deploy-pages.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## What this is

SlideCraft HTML is a small collection of VS Code GitHub Copilot skills that generate styled HTML pages from visual references or short prompts.

Drop in a mood board — inspiration photos, screenshots, images, sketches, or visual references — ask for an HTML page, and GitHub Copilot Chat produces a self-contained HTML file with matching design tokens.

This repo is for people who want pretty presentation-style HTML without fighting PowerPoint, CSS from scratch, or the tiny goblin in their brain that says, “What if we just use another gradient?”



## Get started

Read the published how-to: <https://AmieDD.github.io/slidecraft-html/>

## What's in this repo

* `.github/skills/` contains the Copilot skill definitions (`moodboard-to-html`, `moodboard-to-template`, `zine-page`).
* `docs/` is the GitHub Pages source; `docs/index.html` is the published how-to page.
* `example-generated-html/` holds sample pages produced by the skills so you can see the target output before you run anything.

## How it works

You can run these skills from this repo, or copy them into an existing repo where you want Copilot to generate styled HTML pages.

For a visual walkthrough, follow the instruction page here: <https://AmieDD.github.io/slidecraft-html/>

1. Open this repo in VS Code, or copy the skill you want from `.github/skills/` into your existing repo.
2. Choose one of the skills from `.github/skills/`.
3. Add a mood board, inspiration photo, screenshot, sketch, or short design prompt to GitHub Copilot Chat.
4. Ask GitHub Copilot Chat to generate a self-contained HTML page.
5. Review, refine, and customize the generated file.

## License

Released under the [MIT License](LICENSE). © 2026 Amie Dansby.
