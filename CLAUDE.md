# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project files

The portfolio lives at the home directory root — not inside this folder:

| File | Purpose |
|---|---|
| `~/portfolio.html` | Everything: all CSS (inline `<style>`), all JS (inline `<script>`), all HTML |

**Do not** introduce npm, a bundler, or a framework. This project is intentionally dependency-free and runs with `python3 -m http.server 8080` or directly as `file://`.

---

## Architecture

### Routing

`setRoute(name)` shows/hides full-page `.page` sections by their `data-route` attribute. All state is in-memory — there is no URL hash. Nav buttons carry `data-route`; the logo always routes to `feed`.

### Feed cards

Cards are `.pin` elements with `data-cat` (for filter tabs) and `data-page` (sheet key). Clicking calls `openSheet(key, srcEl)`. A JS snippet at page load moves any `.sub` inside `.art` out to be a direct child of `.pin` so it overlays below the card on hover.

Filter tabs match `data-filter` against `data-cat`. Visibility changes use a FLIP animation (snapshot positions before/after, then transition with `translate`).

### Sheet panel

`openSheet(key, srcEl)` reads from the `PAGES` object and writes HTML into `#sheetBody`. If `PAGES[key].custom` exists, that string is used verbatim (full custom layout); otherwise the default template renders `eyebrow`, `title`, `lead`, `facts`, a figure placeholder, and `body`.

The app shell gains `.sheet-open` on open, triggering: rail hides, topbar hides, tabs reflow into a fixed header bar, feed shifts left by `--panel-w: 50vw`. A FLIP animation repositions the active pin.

### Experience timeline

The experience section (`#expView`) is a scroll-driven horizontal track. A sticky sentinel + `IntersectionObserver` drives `updateExperienceTrackNow()` which reads scroll position and sets `--exp-track-x` on `.exp-scroll` to translate `.exp-track`. 

End-of-scroll: `endHold = viewport.clientHeight * 0.55` — a buffer where the card deck fades out (smoothstep opacity + translateY) before the sticky viewport releases. Cards individually fade in via `--scroll-reveal-opacity` as they enter the track.

### Theme

`data-theme="dark"` on `<html>` drives all color tokens via CSS custom property overrides in `[data-theme="dark"]`. Toggling adds `.theme-transitioning` for a 520ms crossfade, then removes it.

## Key globals (in `portfolio.html` inline `<script>`)

| Symbol | What it is |
|---|---|
| `PAGES` | Object keyed by card slug. Each entry has `meta`, `eyebrow`, and either `custom` (raw HTML string) or `title/lead/facts/body`. |
| `FIG_BG` | Map of `data-cat` → background color for the placeholder figure in default sheet layouts. |
| `setRoute(name)` | Switches active page view. |
| `openSheet(key, srcEl)` | Populates and opens the sheet panel. |
| `closeSheet()` | Closes the panel and restores layout. |
| `updateExperienceTrackNow()` | Recalculates horizontal scroll position and card reveal state. |

---

## Design tokens (CSS custom properties)

**Color:** `--paper`, `--paper-2`, `--ink`, `--ink-mid`, `--ink-low`, `--line`, `--accent`, `--accent-soft`, `--c-clay`, `--c-blue`, `--c-cream`, `--c-yellow`, `--c-sage`, `--c-plum`, `--c-sand`, `--c-charcoal`

**Motion:** `--motion-fast` (180ms), `--motion-med` (420ms), `--motion-slow` (480ms), `--ease-out` (cubic-bezier 0.22,1,0.36,1), `--ease-soft` (cubic-bezier 0.19,1,0.22,1)

**Fonts:** `--serif` (Instrument Serif), `--sans` / `--mono` (Inter)

**Layout:** `--panel-w: 50vw` (sheet panel width)

---

## CSS class conventions

| Prefix | Used for |
|---|---|
| `cs-` | Case study sheet content components |
| `ab-` | About page sheet content |
| `exp-` | Experience timeline components |
| `art-*` | Feed card color variants (`art-clay`, `art-sage`, `art-charcoal`, etc.) |
| `pin` | Feed card (`.pin`, `.pin-active`, `.pin-meta`, `.sub`) |
