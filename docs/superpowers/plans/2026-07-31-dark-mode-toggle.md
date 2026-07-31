# Dark Mode Toggle Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a light/dark theme toggle to `index.html`, matching the mechanism already used on the sibling Artist portfolio site.

**Architecture:** CSS custom properties defined in `:root`, overridden by `@media (prefers-color-scheme: dark)`, overridden again by explicit `:root[data-theme="light"|"dark"]` blocks. An inline `<script>` in `<head>` sets `data-theme` before first paint from `localStorage`/system preference. A nav icon button flips the attribute on click and persists the choice.

**Tech Stack:** Plain HTML/CSS/JS, no build step, no test framework — same constraints as the rest of this site. "Tests" in this plan are concrete manual browser-verification steps, not automated tests.

## Global Constraints

- No build tooling, no external dependencies — everything stays inline in `index.html` (per spec "Out of scope / unchanged").
- Dark palette values must be exactly as specified in the design spec (`docs/superpowers/specs/2026-07-31-dark-mode-toggle-design.md`).
- The hero, `.quote-section`, and `footer` are NOT touched — they already read correctly in both themes.
- Toggle icon markup/behavior must match the Artist site's sun/moon SVG pair and click handler for consistency across both personal sites.

---

### Task 1: Add `--surface` variable and dark theme CSS blocks

**Files:**
- Modify: `index.html` (`:root` block, currently lines 8–22; insert new blocks immediately after)

**Interfaces:**
- Produces: CSS custom properties `--ink`, `--ink-soft`, `--paper`, `--paper-alt`, `--surface` (new), `--line`, `--accent-tint`, `--accent`, `--accent-dark` — each with a light value in `:root` and a dark value in both the `prefers-color-scheme` media query and the `data-theme` attribute selectors. Later tasks (Task 2) rely on `--surface` existing.

- [ ] **Step 1: Add `--surface: #FFFFFF;` to the existing `:root` block**

In `index.html`, inside the `:root{...}` block (around line 20, after `--line: #E6E3F2;`), add:

```css
    --surface: #FFFFFF;
```

- [ ] **Step 2: Add the dark-mode `@media` override block**

Immediately after the closing `}` of the `:root` block, insert:

```css
  @media (prefers-color-scheme: dark){
    :root{
      --ink: #F1EFFA;
      --ink-soft: #B8B4CC;
      --paper: #100F1B;
      --paper-alt: #17162A;
      --surface: #1E1D30;
      --line: #2C2A42;
      --accent-tint: #2A2650;
      --accent: #9C8CFF;
      --accent-dark: #C9C1FF;
    }
  }
```

- [ ] **Step 3: Add explicit `data-theme` override blocks**

Immediately after the `@media` block from Step 2, insert:

```css
  :root[data-theme="light"]{
    --ink: #1B1B23;
    --ink-soft: #52515E;
    --paper: #FAFAFC;
    --paper-alt: #F1EEFB;
    --surface: #FFFFFF;
    --line: #E6E3F2;
    --accent-tint: #EFECFE;
    --accent: #6C5CE7;
    --accent-dark: #4B3FC4;
  }

  :root[data-theme="dark"]{
    --ink: #F1EFFA;
    --ink-soft: #B8B4CC;
    --paper: #100F1B;
    --paper-alt: #17162A;
    --surface: #1E1D30;
    --line: #2C2A42;
    --accent-tint: #2A2650;
    --accent: #9C8CFF;
    --accent-dark: #C9C1FF;
  }
```

- [ ] **Step 4: Replace hardcoded `#fff`/`white` backgrounds with `var(--surface)`**

Three replacements in `index.html`:
- Line ~94: `border: 6px solid #fff; outline: 1px solid var(--line);` → `border: 6px solid var(--surface); outline: 1px solid var(--line);`
- Line ~120: `background:#fff; border: 1px solid var(--line); border-radius: var(--radius);` (in `.card`) → `background: var(--surface); border: 1px solid var(--line); border-radius: var(--radius);`
- Line ~137: `border: 1px solid var(--line); border-radius: var(--radius);\n    padding: 24px; background: #fff;` (in `.fact`) → same, with `background: var(--surface);`

- [ ] **Step 5: Add the theme-init script to `<head>`**

Immediately before the closing `</head>` tag, add:

```html
<script>
  (function(){
    var stored = null;
    try{ stored = localStorage.getItem('theme'); }catch(e){}
    var prefersDark = window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches;
    var theme = stored || (prefersDark ? 'dark' : 'light');
    document.documentElement.setAttribute('data-theme', theme);
  })();
</script>
```

- [ ] **Step 6: Manual verification — no toggle yet, but page must still render correctly**

Open `index.html` directly in a browser. Expected: page looks identical to before (light theme, since no stored preference and assuming a light-mode OS/browser default, or matches your OS dark/light setting if already in dark mode — either way, no visual regression, no console errors). Open browser dev tools console: confirm no JS errors.

- [ ] **Step 7: Commit**

```bash
cd "d:/Work/Persona"
git add index.html
git commit -m "Add dark mode CSS variables and theme-init script"
```

---

### Task 2: Add the toggle button and click handler

**Files:**
- Modify: `index.html` (`.topnav` markup, currently around lines 203–210; end-of-body `<script>` block)

**Interfaces:**
- Consumes: `data-theme` attribute set by Task 1's init script; `--surface`/dark variable set from Task 1.
- Produces: button `#themeToggle` with working click handler; no further tasks depend on this (final task in this plan).

- [ ] **Step 1: Add toggle button styles**

In the `<style>` block, after the `.langswitch` rules (around line 63, after `.langswitch button.active{...}`), add:

```css
  .theme-toggle{
    display:flex; align-items:center; justify-content:center;
    width: 36px; height: 36px;
    border: 1px solid rgba(255,255,255,0.35);
    border-radius: 50%;
    background: rgba(255,255,255,0.08);
    color: rgba(255,255,255,0.85);
    cursor:pointer; padding:0;
    transition: background 0.25s ease, color 0.25s ease;
  }
  .theme-toggle:hover{ color:#fff; background: rgba(255,255,255,0.18); }
  .theme-toggle svg{ width:18px; height:18px; }
  .theme-toggle .icon-moon{ display:none; }
  :root[data-theme="dark"] .theme-toggle .icon-sun{ display:none; }
  :root[data-theme="dark"] .theme-toggle .icon-moon{ display:block; }
```

- [ ] **Step 2: Add the button markup to `.topnav`**

Find the `.topnav` div (around lines 203–210). It currently ends with the `.langswitch` `nav`/div structure. Add the toggle button as a sibling right after the closing `</nav>` (or after the language switch div, whichever is last child of `.topnav`) — read the actual current markup first to place it correctly, then insert:

```html
      <button type="button" class="theme-toggle" id="themeToggle" aria-label="Switch to dark theme" aria-pressed="false">
        <svg class="icon-sun" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line></svg>
        <svg class="icon-moon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg>
      </button>
```

- [ ] **Step 3: Add the click handler script**

Find the existing end-of-body `<script>` block (the one handling language switching / other page JS, if any — if none exists yet, add a new `<script>` immediately before `</body>`). Add:

```html
<script>
  var themeToggle = document.getElementById('themeToggle');
  function applyThemeButtonState(theme){
    var isDark = theme === 'dark';
    themeToggle.setAttribute('aria-pressed', isDark);
    themeToggle.setAttribute('aria-label', isDark ? 'Switch to light theme' : 'Switch to dark theme');
  }
  applyThemeButtonState(document.documentElement.getAttribute('data-theme'));
  themeToggle.addEventListener('click', function(){
    var current = document.documentElement.getAttribute('data-theme');
    var next = current === 'dark' ? 'light' : 'dark';
    document.documentElement.setAttribute('data-theme', next);
    try{ localStorage.setItem('theme', next); }catch(e){}
    applyThemeButtonState(next);
  });
</script>
```

- [ ] **Step 4: Manual verification — toggle behavior**

Open `index.html` in a browser:
1. Confirm the sun/moon icon button appears in the top nav, to the right of (or near) the language switch.
2. Click it. Expected: page immediately switches between light and dark palettes (backgrounds, text, card/fact surfaces, badges all change color); icon swaps between sun and moon; no console errors.
3. Reload the page. Expected: the theme you last selected persists (does not reset to system default).
4. Open dev tools → Application → Local Storage: confirm a `theme` key with value `light` or `dark` matching the current state.
5. Scroll through About, What I Do, Facts, and Connect sections in both themes: confirm all text remains legible (sufficient contrast) against its background in both themes.
6. Confirm the hero image, quote section, and footer look unchanged from before this feature (still dark/photo-based, unaffected by the toggle).

- [ ] **Step 5: Commit**

```bash
cd "d:/Work/Persona"
git add index.html
git commit -m "Add dark mode toggle button to nav"
```

---

### Task 3: Push to GitHub

**Files:** none (git operation only)

**Interfaces:** none

- [ ] **Step 1: Push the commits**

```bash
cd "d:/Work/Persona"
git push
```

- [ ] **Step 2: Verify**

Run `git log --oneline -5` and confirm the two feature commits from Tasks 1–2 are present and `git status` shows the branch up to date with `origin/main`.
