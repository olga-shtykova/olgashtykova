# Dark Mode Toggle — Design

## Purpose

Add a light/dark theme toggle to the Persona site (`index.html`), matching the
mechanism already implemented on the sibling Artist portfolio site, so both
personal sites behave consistently.

## Mechanism

- `<html>` carries a `data-theme="light"|"dark"` attribute.
- An inline `<script>` in `<head>` sets this attribute before first paint:
  reads `localStorage.getItem('theme')`, falling back to
  `window.matchMedia('(prefers-color-scheme: dark)')` if nothing is stored.
  This avoids a flash of the wrong theme on load.
- CSS custom properties are defined three times:
  1. Light defaults in `:root`.
  2. A `@media (prefers-color-scheme: dark)` override (covers users who
     never touch the toggle).
  3. Explicit `:root[data-theme="light"]` / `:root[data-theme="dark"]`
     blocks, which win over the media query once the user has an explicit
     preference.
- A sun/moon icon toggle button is added to `.topnav`, next to the existing
  language switch. Clicking it flips `data-theme` on `<html>` and persists
  the choice to `localStorage`. Same SVG icon pair as the Artist site.

## New/changed CSS variables

A `--surface` variable is introduced to replace the hardcoded `#fff`
backgrounds currently used in `.about-photo` (border), `.card`, and `.fact`.
All other existing variables (`--ink`, `--ink-soft`, `--paper`,
`--paper-alt`, `--line`, `--accent-tint`, `--accent`, `--accent-dark`) get
dark equivalents.

| Variable | Light (current) | Dark (new) |
|---|---|---|
| `--ink` | `#1B1B23` | `#F1EFFA` |
| `--ink-soft` | `#52515E` | `#B8B4CC` |
| `--paper` | `#FAFAFC` | `#100F1B` |
| `--paper-alt` | `#F1EEFB` | `#17162A` (reuses existing `--night-soft`) |
| `--surface` *(new)* | `#FFFFFF` | `#1E1D30` |
| `--line` | `#E6E3F2` | `#2C2A42` |
| `--accent-tint` | `#EFECFE` | `#2A2650` |
| `--accent` | `#6C5CE7` | `#9C8CFF` |
| `--accent-dark` | `#4B3FC4` | `#C9C1FF` (matches the quote section's existing cite color) |

## Out of scope / unchanged

- The hero section, `.quote-section`, and `footer` are already fixed
  dark/photo-based sections in both themes today — no change needed, they
  read fine under either theme.
- No build step, no new files beyond this spec — everything stays inline in
  `index.html`, consistent with how this site is currently structured.

## Testing

Manual verification in a browser: toggle switches instantly, persists
across reload, respects system preference when no stored choice exists, and
all sections (about, work, facts, connect) remain readable/legible in both
themes.
