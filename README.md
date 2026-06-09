# TAMZ — Master Platform

**One platform. Two visual experiences. Zero build tools.**

TAMZ is a complete GATE CSE 2027 + placement prep hub delivered as three plain HTML files. Users choose between two fully distinct interfaces — an elegant editorial dashboard or a hacker-aesthetic terminal — and their preference is remembered across sessions. Both themes present identical tools, content, and data; only the visual layer differs.

---

## Table of Contents

1. [Project Structure](#project-structure)
2. [File Inventory](#file-inventory)
3. [How It Works](#how-it-works)
4. [Theme System](#theme-system)
5. [Landing Page](#landing-page)
6. [Theme Switcher Widget](#theme-switcher-widget)
7. [Auto-Restore Logic](#auto-restore-logic)
8. [Executive Theme — Feature Reference](#executive-theme--feature-reference)
9. [Hacker Theme — Feature Reference](#hacker-theme--feature-reference)
10. [Tools & Linked Modules](#tools--linked-modules)
11. [Keyboard Shortcuts](#keyboard-shortcuts)
12. [Deployment](#deployment)
13. [Customisation Guide](#customisation-guide)
14. [Browser Compatibility](#browser-compatibility)
15. [Design Decisions](#design-decisions)

---

## Project Structure

```
tamz/
├── index.html        ← Landing page / theme router
├── executive.html    ← Premium editorial dashboard (Theme 1)
├── hacker.html       ← Terminal cyber mode (Theme 2)
└── README.md         ← This file
```

That's the entire platform. No npm, no bundler, no build step, no dependencies to install. Drop the four files on any static host and it's live.

---

## File Inventory

| File | Size | Role | Original source |
|---|---|---|---|
| `index.html` | ~16 KB | Landing page, theme selection, auto-redirect router | New (created for master platform) |
| `executive.html` | ~83 KB | Premium dark/light editorial dashboard | `index__1_.html` (preserved) |
| `hacker.html` | ~71 KB | Terminal / hacker aesthetic dashboard | `index.html` (preserved) |

Both `executive.html` and `hacker.html` are the original production files with two surgical injections each: an auto-restore guard script and a floating theme-switcher widget. No original code, styles, layout, or functionality was modified.

---

## How It Works

The platform uses three cooperating mechanisms:

**1. localStorage preference key**

```js
localStorage.setItem("tamz-theme", "executive") // or "hacker"
```

This key is the single source of truth. Every file reads it on load; every theme change writes to it.

**2. Auto-restore redirect**

Each theme file checks the key at the very top of its `<body>` (before any rendering). If the stored preference points to the other theme, it immediately calls `window.location.replace()` to redirect. The user never sees a flash of the wrong theme.

**3. Theme switcher widget**

A floating `⊞` button fixed to the bottom-right corner of each theme allows switching at any time. It writes the new preference to localStorage and navigates to the correct file.

The landing page (`index.html`) is only visited when no preference is stored, or when the user explicitly clicks "Choose again."

---

## Theme System

### Theme 1 — Executive / Premium Mode (`executive.html`)

A luxury editorial dashboard. Inspired by high-end editorial publishing — warm amber and cream palette, Cormorant Garamond serif for headings, Syne sans-serif for UI. Supports both light and dark sub-modes with a toggle in the navbar.

**Visual identity:**
- Colors: `#f7f5f0` light background / `#0c0b08` dark background, `#c17b3a` amber accent
- Typography: Cormorant Garamond (display) + Syne (UI)
- Motion: subtle `fadeUp` entrance animations, `translateY(-2px)` card hovers, smooth `0.2s` transitions

### Theme 2 — Hacker / Terminal Mode (`hacker.html`)

A cyber terminal aesthetic. Matrix rain canvas background, CRT scanlines overlay, radial vignette, and a full interactive bash emulator. Monochrome green-on-black with cyan, amber, and purple accents for semantic colour coding.

**Visual identity:**
- Colors: `#020b02` background, `#00ff41` primary green, `#00d4ff` cyan, `#ffb000` amber
- Typography: JetBrains Mono (primary) + Share Tech Mono (ASCII/display)
- Motion: matrix rain (canvas), boot sequence, text glow, pulse rings, glitch effect

---

## Landing Page

`index.html` is a purpose-built theme chooser. It appears only when no `tamz-theme` preference is stored in localStorage, or when the user navigates back to it from within a theme.

**What it does:**

On load, it immediately reads `localStorage.getItem("tamz-theme")`. If a value of `"hacker"` or `"executive"` is found, it calls `window.location.href` to the correct theme file before rendering the selection UI. Return visits therefore skip the landing page entirely.

If no preference is found, it renders a full-screen selector with:

- Matrix rain canvas background matching the hacker theme's visual identity
- CRT scanline and vignette overlays
- A staggered fade-in entrance sequence (8 animation steps, 0.3s to 1.9s delays)
- Two side-by-side theme cards, each styled in its own visual language
- A smooth `0.35s` fade-to-black transition cover before redirecting

**Selecting a theme:**

```js
function selectTheme(theme) {
  card.classList.add("selected");               // flash animation
  localStorage.setItem("tamz-theme", theme);    // save preference
  transitionCover.classList.add("active");       // fade to black
  setTimeout(() => {
    window.location.href = theme === "hacker"
      ? "hacker.html"
      : "executive.html";
  }, 350);
}
```

Cards are also keyboard accessible — `Enter` or `Space` triggers selection.

---

## Theme Switcher Widget

Both `executive.html` and `hacker.html` include a floating switcher injected just before their `</body>` tag. The widget is self-contained: its own `<style>` block, its own `<div>` structure, and an IIFE for all logic — nothing interferes with the host page's existing JavaScript or CSS.

**Structure:**

```
#tamz-switcher (fixed, bottom-right)
  ├── #tamz-switcher-menu (hidden by default)
  │     ├── .tsw-label       "// tamz theme" / "Interface"
  │     ├── .tsw-btn.active  Current theme (non-clickable)
  │     ├── .tsw-btn         Alternate theme (click to switch)
  │     ├── .tsw-sep         Divider
  │     └── .tsw-back        "← choose again" → index.html
  └── #tamz-switcher-toggle  The ⊞ button
```

**Behaviour:**

- Clicking `⊞` toggles the menu open/closed.
- Clicking anywhere outside the widget closes the menu.
- Clicking the alternate theme button writes to localStorage and navigates to the other file.
- Clicking "← choose again" navigates to `index.html`, which will show the chooser since neither theme will auto-redirect back.
- Keyboard shortcut **Alt+T** opens/closes the menu from anywhere on the page.

**Styling philosophy:**

The switcher in `hacker.html` uses the terminal's own CSS variables (`#020b02` background, `rgba(0,255,65,0.35)` borders, `var(--mono)` font) so it looks native to that environment. The switcher in `executive.html` uses the executive theme's CSS variables (`var(--surface)`, `var(--border)`, `var(--accent)`, rounded `8px` corners) so it blends with that environment. Neither switcher imports any external styles.

---

## Auto-Restore Logic

Each theme file has a small inline `<script>` injected near the top of `<body>`. It runs before the DOM is painted:

**In `hacker.html`:**
```js
(function() {
  var t = localStorage.getItem("tamz-theme");
  if (t === "executive") {
    window.location.replace("executive.html");
  }
})();
```

**In `executive.html`:**
```js
(function() {
  var t = localStorage.getItem("tamz-theme");
  if (t === "hacker") {
    window.location.replace("hacker.html");
  }
})();
```

`window.location.replace()` is used instead of `href` assignment so the redirect does not add an entry to the browser's history stack. The user pressing Back will go to whatever page they came from, not bounce between themes.

**Why this placement matters:** The script runs synchronously before the rest of the body parses. The hacker theme's boot animation, matrix rain, and heavy font loading never begin if a redirect is triggered — no flash, no wasted render work.

---

## Executive Theme — Feature Reference

### Navigation
- **Sticky navbar** with glass blur (`backdrop-filter: blur(22px)`) and border separation.
- **Sidebar** (right-side drawer) with overlay. Opens via hamburger button. Contains all 11 tool links grouped by category. Closes on overlay click or the × button.
- **Dark/light toggle** (`#themeBtn`): switches `[data-theme]` attribute on `<html>` between `"light"` and `"dark"`. Preference is stored under the `dsaTheme` localStorage key (separate from the inter-theme switcher key). Uses `prefers-color-scheme` as fallback on first visit.

### Countdown Bar
- Live countdown to GATE 2027 exam date (1 February 2027, 09:30 IST).
- Updates every second via `setInterval`.
- Progress track bar at the bottom fills proportionally based on elapsed time since May 2026 start.
- Phase chip shows current phase and week number.

### Hero Section
- Two-column grid: left side has heading, description, CTA buttons, and stats bar; right side has a panel of six quick-link cards (`hc` component).
- Stats bar shows: 1,492 PYQs indexed / 260+ DSA problems / 16 GATE subjects / Feb '27 exam.
- On viewports below 920px, the right panel and stats bar hide; layout collapses to single column.

### Week Snapshot
- Displays the current week's study focus.
- Week number, date range, and phase chip are dynamically calculated from a hardcoded start date (4 May 2026) using `Date` arithmetic.
- Split two-column layout: GATE topic (left) + DSA topic (right).

### Tool Cards (`card` component)
- Rounded `12px` border-radius cards with a coloured top border accent.
- Hover state: `translateY(-2px)` lift + `box-shadow` + border colour change + glow blob (`.card-glow`) fades in.
- Each card uses CSS custom properties (`--c`, `--cg`) for per-card accent theming without extra classes.

### Roadmap Phases
- Five phase rows showing name, week range, subject tags, completion percentage, and animated progress bar.
- Progress bars animate from 0 to their target width on page load via `setTimeout` + inline `style.width`.

### Subject Grid
- 16 GATE CSE subjects rendered as compact link items.
- Each links to the relevant resource (Notes Hub or subject-specific PYQ set).

### DSA Grid
- 35-week problem set cards, one per week, coloured by phase.
- Each links to the DSA Bank filtered to that week.

### Command Palette
- Opens with `Ctrl+K` / `Cmd+K` or the search button in the navbar.
- Fuzzy search across all 11 tools (name, description, tag, group).
- Arrow key navigation + Enter to open the highlighted item.
- Highlights matching text in results using a `<mark>` injection with regex.
- Backdrop click or `Escape` to close.

### Ambient Sound Mode
- Button in navbar (music note icon).
- Activates a pulsing ring animation on the button and shifts the site accent to teal.
- A toast notification slides up from the bottom with mode controls.
- Mode options: Rain / Cafe / White Noise (UI is present; audio source hookup is left to the developer).

---

## Hacker Theme — Feature Reference

### Boot Sequence
- Full-screen overlay (`#boot-screen`) with ASCII TAMZ logo, progress bar, and status messages.
- Boot messages cycle through system initialisation strings over ~2.5 seconds.
- Bar fills in discrete jumps. On completion, the overlay fades out (`opacity: 0`, `pointer-events: none`) and the main content becomes interactive.
- ASCII art uses the Share Tech Mono typeface.

### Matrix Rain
- HTML5 Canvas (`#matrix-canvas`) fixed to the viewport, `z-index: 0`, `opacity: 0.045`.
- Renders Japanese katakana + hex characters falling in columns 16px wide.
- Each column starts at a random negative Y position. When a column exceeds viewport height, it resets with a 3% probability per frame (creates natural variation in column lengths).
- Draws at 20fps (50ms interval) with a semi-transparent black fill each frame for the trail fade effect.

### CRT Effects
- `body::before`: `repeating-linear-gradient` scanlines, 2px transparent + 2px `rgba(0,0,0,0.08)`, full-screen fixed overlay.
- `body::after`: radial gradient vignette darkening from 60% radius to the edges.
- Both overlays use `pointer-events: none` so they do not block interaction.

### Top Bar
- macOS-style traffic-light dots (red/yellow/green).
- Centred title with `text-shadow` glow.
- Right side: live pulse indicator, live clock (HH:MM:SS updated every second), current week indicator.
- Sticky via `position: sticky; top: 0; z-index: 100`.

### Countdown Strip
- Horizontal scrollable strip (overflow-x: auto for mobile) with days/hrs/min/sec counters.
- Phase badge on the right.
- Two-pixel progress bar below the strip fills proportionally to study plan elapsed time.

### Hero Terminal Output
- Simulates a shell command being typed via a character-by-character interval (`typeHero` function).
- After typing completes, the output block fades in showing ASCII art + system info grid.
- ASCII TAMZ banner rendered in Share Tech Mono.

### Interactive Terminal (`#ltInput` / `#ltOutput`)
- Full bash-emulator interface with `>` prompt.
- **Command history**: Up/Down arrows cycle through previously entered commands (`cmdHistory` array, `histIdx` pointer).
- **Tab autocomplete**: matches input prefix against a predefined command list. Single match: auto-completes. Multiple matches: prints them as suggestions.
- **Ctrl+K** focuses the terminal input and scrolls it into view.
- Clicking anywhere on the page (except links, cards, and phase rows) auto-focuses the terminal input.

**Available commands:**

| Command | What it does |
|---|---|
| `help` | Lists all available commands in a formatted grid |
| `ls` | Lists all 11 mounted modules as a table with URLs |
| `status` | Shows current week, phase, subject, DSA target, and GATE countdown |
| `roadmap` | Prints all 5 phases with week ranges and subject tags |
| `roadmap ai` | Prints AI-specific roadmap phase with schedule detail |
| `open <module>` | Opens the named tool in a new tab (see module names below) |
| `pyq` | Prints PYQ database summary (question count, years, rising topics) |
| `dsa` | Prints DSA Bank summary (problem counts by phase) |
| `trend` | Prints trend analysis summary (rising topics, NAT question stats) |
| `neofetch` | Renders a neofetch-style system info block with ASCII fox art |
| `clear` | Clears the terminal output |
| `cd <section>` | Smooth-scrolls to a page section (roadmap, pyq, exam, dsa, subjects) |

**Open module names:** `roadmap`, `os`, `dbms`, `cn`, `notes`, `dsa`, `ai`, `intelligence`, `pyq`, `trend`, `cbt`, `subjectcbt`, `hub`, `resources`

### Tool Cards (`tool-card` component)
- Dark `var(--bg2)` background, 1px green border.
- Left 2px accent strip changes colour based on `--c` CSS variable.
- Hover: `translateX(3px)` slide + border colour brightens + background tints.
- Status badge (MOUNTED / LIVE) in the top-right of each card.
- Pills row for tags.

### Phase Rows
- Horizontal flex rows with large phase number on the left, info in the centre, progress bar on the right.
- Phase number has `text-shadow` glow in its accent colour.
- Animated progress bar fills on page load.
- Arrow (`→`) in far right transitions to the phase colour on hover.

### Glitch Effect
- CSS `@keyframes glitch` clips the element at random vertical slices and applies small X translations.
- Applied via `.glitch` class with `::before` (cyan tint) and `::after` (red tint) pseudo-elements at reduced opacity.
- Used on the TAMZ title in certain states.

---

## Tools & Linked Modules

Both themes link to the same eleven external tools. All open in `target="_blank"`.

| Module | URL | Category |
|---|---|---|
| GATE + Placement Roadmap | `gatetamz.vercel.app` | Study |
| GATE CSE Notes Hub | `gatenote.vercel.app` | Study |
| DSA Question Bank | `dsatamz.vercel.app` | Study |
| GATE CSE Hub | `gatecsehub.vercel.app` | Study |
| Resource Links | `gatetamzresources.vercel.app` | Study |
| AI Tutor | `tahagateai.vercel.app` | PYQ & Intelligence |
| GATE Intelligence — CSE Bank | `gateecotaha.vercel.app` | PYQ & Intelligence |
| PYQ Question Bank | `gatepvq.vercel.app` | PYQ & Intelligence |
| Trend Analysis | `tahagatetrend.vercel.app` | PYQ & Intelligence |
| GATE CBT Simulator | `tahagatecbt.vercel.app` | Exam Simulation |
| Subject CBT Simulator | `tahagatesubjectcbt.vercel.app` | Exam Simulation |

---

## Keyboard Shortcuts

### Global (both themes)
| Shortcut | Action |
|---|---|
| `Alt+T` | Open / close the TAMZ theme switcher widget |

### Executive theme
| Shortcut | Action |
|---|---|
| `Ctrl+K` / `Cmd+K` | Open command palette |
| `↑` / `↓` | Navigate command palette results |
| `Enter` | Open highlighted result |
| `Escape` | Close command palette |
| `A` | Toggle ambient sound mode |

### Hacker theme
| Shortcut | Action |
|---|---|
| `Ctrl+K` / `Cmd+K` | Focus terminal input |
| `↑` | Previous command (history) |
| `↓` | Next command (history) |
| `Tab` | Autocomplete command |
| `Enter` | Execute command |

---

## Deployment

### Vercel (recommended)

```bash
# Install Vercel CLI if you haven't
npm i -g vercel

# From the project folder
vercel
```

Vercel auto-detects static HTML. No configuration file needed. All three files will be served from the same domain, which is required for the `localStorage` theme preference to work across files.

### Netlify

Drag and drop the project folder onto [app.netlify.com/drop](https://app.netlify.com/drop). Done.

### GitHub Pages

```bash
git init
git add .
git commit -m "initial"
gh repo create tamz --public --push --source=.
# Then enable Pages in repo Settings → Pages → Deploy from branch: main / (root)
```

### Any static host (nginx, Apache, S3, Cloudflare Pages)

Upload all four files to the same directory. Ensure the server is configured to serve `.html` files with `Content-Type: text/html`. No rewrites or redirects are required.

**Critical requirement:** All three HTML files must be served from the **same origin** (same protocol + domain + port). localStorage is origin-scoped; if the files are on different domains, the preference key will not be shared and theme switching will not work.

---

## Customisation Guide

### Changing the GATE exam date

Both themes compute the countdown from the exam date. Search for the date string in each file:

In `hacker.html`:
```js
// Search for: new Date(2027, 1, 1, 9, 30, 0)
// Month is 0-indexed, so 1 = February
```

In `executive.html`:
```js
// Search for the same pattern near the countdown setInterval
```

Update both files to your exam date.

### Changing the current week / phase

In both files, search for `W5` (the current week marker) and `Phase 1` to update the active indicators. The week snapshot section in `executive.html` is calculated dynamically from a start date — search for `"4 May 2026"` or similar to update the base date.

### Adding a new tool

In both themes, tools appear in multiple places. For a complete addition you need to update:

**In `executive.html`:**
- The sidebar `<div class="sidebar">` section (add a new `<a class="sb-link">`)
- The appropriate section card grid (add a new `<a class="card">`)
- The command palette `ITEMS` array in the `<script>` block at the bottom

**In `hacker.html`:**
- The tool grid `<div class="tool-grid">` section (add a new `<a class="tool-card">`)
- The `COMMANDS.ls` function return value (add a row to the table output)
- The `COMMANDS.open` handler's `TOOLS` object (add a new key with `name` and `url`)
- The Tab autocomplete `allCmds` array (add `"open yourmodule"`)

### Restyling the theme switcher

The switcher in each file is a self-contained block starting with `<!-- TAMZ THEME SWITCHER -->`. All its styles are scoped to `#tamz-switcher` and `.tsw-*` class names and are unlikely to conflict with the host page. Edit the `<style>` block within that section freely.

### Adjusting the landing page

`index.html` is standalone and only references fonts from Google Fonts. All colours are defined as CSS custom properties in `:root`. The matrix rain density and speed are controlled by the `cols` calculation and the `50` ms interval in the canvas script — increase the interval number to slow it down.

---

## Browser Compatibility

| Feature | Chrome | Firefox | Safari | Edge |
|---|---|---|---|---|
| CSS custom properties | ✓ | ✓ | ✓ | ✓ |
| `backdrop-filter` | ✓ | ✓ (with flag off by default until v103) | ✓ | ✓ |
| Canvas 2D | ✓ | ✓ | ✓ | ✓ |
| `localStorage` | ✓ | ✓ | ✓ | ✓ |
| CSS `position: sticky` | ✓ | ✓ | ✓ | ✓ |
| `prefers-color-scheme` media query | ✓ | ✓ | ✓ | ✓ |
| Google Fonts (JetBrains Mono, Cormorant Garamond, Syne) | Requires internet | Requires internet | Requires internet | Requires internet |

**Minimum recommended versions:** Chrome 88+, Firefox 89+, Safari 14+, Edge 88+.

**Private browsing / incognito:** `localStorage` is available but cleared when the session ends. Users in private mode will see the landing page on every visit.

**JavaScript disabled:** The pages will render their HTML structure but the countdown, terminal, boot animation, matrix rain, theme switcher, and auto-restore redirect all require JavaScript. There is no `<noscript>` fallback.

---

## Design Decisions

**Why three separate HTML files instead of a React SPA?**

The original files are complete, self-contained production documents with tightly coupled HTML, CSS, and JavaScript. Decomposing them into components would require understanding and replicating every CSS specificity rule, every animation keyframe, every JS closure — and would risk introducing regressions. Three static files with a thin routing layer between them is safer, faster to load (no framework overhead), and easier to debug. The "architecture" is the browser's own navigation model.

**Why localStorage instead of URL parameters or cookies?**

localStorage persists across tabs and sessions without server involvement, requires no cookie consent considerations, and is accessible from plain JavaScript in any context. URL parameters would require server-side routing or a JS router to intercept. Cookies would work but add unnecessary complexity for a client-only preference.

**Why `window.location.replace()` for the auto-restore redirect instead of `window.location.href`?**

`replace()` does not push an entry onto the browser history stack. This means pressing the Back button after an auto-restore redirect takes the user to wherever they came from (e.g., a link shared externally), not into an infinite redirect loop between the two theme files.

**Why inject the switcher before `</body>` rather than in `<head>`?**

The switcher contains DOM elements that need to be appended to `<body>`. Placing it at the end of the body ensures the rest of the page's DOM is already constructed before the switcher's IIFE runs, avoiding any need for `DOMContentLoaded` event listeners. It also means the switcher is the last thing painted — it sits visually above everything else naturally.

**Why is the auto-restore script placed near the top of `<body>` rather than in `<head>`?**

Scripts in `<head>` that call `window.location.replace()` work fine, but both theme files have Google Fonts `<link>` tags and inline `<style>` blocks in `<head>` that would begin loading before the redirect fires. Placing the redirect script as the first element of `<body>` (before any canvas, layout, or animation code) aborts those resource loads earlier, reducing wasted network requests on redirect.
