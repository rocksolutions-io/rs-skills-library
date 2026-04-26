---
name: web-app-bootstrapper
description: >
  Bootstrap a new standalone HTML/CSS/JS web app with Rockson's preferred structure, design system,
  and boilerplate — without re-explaining preferences. Generates a sidebar layout with Claude-inspired
  warm brown/cream color tokens, Lucide icons, light/dark mode, and mobile-first CSS.
  Use this skill whenever the user wants to start a new web app, web tool, web interface,
  or web page — including phrases like "build a web app", "scaffold a web project",
  "create a web tool", "start a new project" in a web context.
---

# Web App Bootstrapper

Rockson is a theatre technical director and venue designer who builds standalone web tools for his work — things like equipment trackers, layout planners, spec calculators, and reference tools. He comes from Flutter/Dart and does not know web dev conventions, so you handle all structural decisions. Never ask about file structure, build tools, or design preferences — those are already decided below.

## Step 1: Ask one quick question

Before generating anything, ask:

> **What does this app do?** (One sentence is fine)

That's all you need. Layout, icons, colors, and structure are already decided.

---

## Project Structure

Generate this in the current directory (or a named subfolder if it makes more sense):

```
<app-name>/
├── index.html
├── css/
│   └── style.css
├── js/
│   └── app.js
└── assets/
    └── (empty — for future images)
```

---

## Icons

Use **Lucide Icons** via CDN — clean, minimal SVG icons, no install needed.

Add to `<head>` in `index.html`:
```html
<script src="https://unpkg.com/lucide@latest/dist/umd/lucide.min.js"></script>
```

Add at the end of `<body>` (after app.js):
```html
<script>lucide.createIcons();</script>
```

Use icons like this in HTML:
```html
<i data-lucide="moon"></i>       <!-- dark mode toggle -->
<i data-lucide="sun"></i>        <!-- light mode -->
<i data-lucide="settings"></i>   <!-- settings -->
<i data-lucide="layers"></i>     <!-- layers / stacking -->
<i data-lucide="ruler"></i>      <!-- measurements -->
<i data-lucide="lightbulb"></i>  <!-- lighting -->
```

Browse all icons at: lucide.dev

After any dynamic DOM update that adds new `data-lucide` elements, call `lucide.createIcons()` again to render them.

---

## Design System (CSS Custom Properties)

Inspired by Claude's warm brown/cream UI. Think of this as Flutter's `ThemeData` — defined once, used everywhere via `var(--token-name)`.

```css
/* ===== LIGHT MODE (default) ===== */
:root {
  /* Backgrounds */
  --bg-primary:    #faf8f5;   /* warm cream — main canvas */
  --bg-secondary:  #f0ece5;   /* warm beige — sidebar */
  --bg-hover:      #e8e0d5;   /* hover state */
  --bg-surface:    #ffffff;   /* cards, modals */

  /* Text */
  --text-primary:   #2d2a25;  /* warm near-black */
  --text-secondary: #6b6560;  /* subdued label */
  --text-muted:     #9b958e;  /* placeholder, disabled */

  /* Accent — Claude orange-amber */
  --accent:         #d97757;
  --accent-hover:   #c4603e;
  --accent-subtle:  #f4e4da;  /* light accent bg for badges/tags */

  /* Borders */
  --border:         #e3dbd0;
  --border-strong:  #ccc4b8;

  /* Spacing — multiples of 4px, like Flutter spacing */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;

  /* Shape */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;

  /* Elevation */
  --shadow-sm: 0 1px 3px rgba(45, 42, 37, 0.08);
  --shadow-md: 0 4px 12px rgba(45, 42, 37, 0.12);
}

/* ===== DARK MODE ===== */
[data-theme="dark"] {
  --bg-primary:    #1a1816;
  --bg-secondary:  #222019;
  --bg-hover:      #2d2b24;
  --bg-surface:    #252320;
  --text-primary:  #ede8e0;
  --text-secondary:#a09890;
  --text-muted:    #6b6560;
  --accent:        #e8956d;
  --accent-hover:  #f0a882;
  --accent-subtle: #3d2a1e;
  --border:        #37332d;
  --border-strong: #4a4540;
  --shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.3);
  --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.4);
}
```

---

## Base Styles

```css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  font-size: 14px;
  color: var(--text-primary);
  background: var(--bg-primary);
  transition: background 0.2s, color 0.2s;
}

body {
  min-height: 100vh;
}

/* Lucide icon sizing */
[data-lucide] {
  width: 16px;
  height: 16px;
  stroke-width: 1.8;
  vertical-align: middle;
}
```

Add Inter font to `<head>`:
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
```

---

## Layout: Sidebar (always)

### HTML Structure

```html
<body>
  <div class="app-layout">

    <aside class="sidebar">
      <div class="sidebar-header">
        <span class="app-title">App Name</span>
      </div>
      <nav class="sidebar-nav">
        <a href="#" class="nav-item active">
          <i data-lucide="layers"></i>
          <span>Section One</span>
        </a>
        <a href="#" class="nav-item">
          <i data-lucide="settings"></i>
          <span>Settings</span>
        </a>
      </nav>
    </aside>

    <main class="main-content">
      <header class="topbar">
        <h1 class="page-title">Page Title</h1>
        <button class="icon-btn" id="themeToggle" title="Toggle dark mode (Cmd+Shift+L)">
          <i data-lucide="moon" id="themeIcon"></i>
        </button>
      </header>
      <div class="content-body">
        <!-- Main content goes here -->
      </div>
    </main>

  </div>
</body>
```

### Sidebar CSS

```css
.app-layout {
  display: flex;
  height: 100vh;
}

/* --- Sidebar --- */
.sidebar {
  width: 220px;
  min-width: 220px;
  background: var(--bg-secondary);
  border-right: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}

.sidebar-header {
  padding: var(--space-4) var(--space-4) var(--space-3);
  border-bottom: 1px solid var(--border);
}

.app-title {
  font-size: 15px;
  font-weight: 600;
  color: var(--text-primary);
}

.sidebar-nav {
  display: flex;
  flex-direction: column;
  padding: var(--space-2);
  gap: 2px;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: var(--space-2);
  padding: var(--space-2) var(--space-3);
  border-radius: var(--radius-sm);
  color: var(--text-secondary);
  text-decoration: none;
  font-size: 13.5px;
  font-weight: 500;
  transition: background 0.15s, color 0.15s;
}

.nav-item:hover {
  background: var(--bg-hover);
  color: var(--text-primary);
}

.nav-item.active {
  background: var(--accent-subtle);
  color: var(--accent);
}

/* --- Main area --- */
.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.topbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: var(--space-3) var(--space-6);
  border-bottom: 1px solid var(--border);
  min-height: 52px;
}

.page-title {
  font-size: 17px;
  font-weight: 600;
}

.content-body {
  flex: 1;
  padding: var(--space-6);
  overflow-y: auto;
}

/* --- Mobile: sidebar stacks above content --- */
@media (max-width: 768px) {
  .app-layout {
    flex-direction: column;
    height: auto;
  }
  .sidebar {
    width: 100%;
    min-width: unset;
    border-right: none;
    border-bottom: 1px solid var(--border);
  }
  .main-content {
    overflow: visible;
  }
}
```

---

## Common UI Components

Use CSS variables throughout — never hardcode colors or sizes.

### Button
```css
.btn {
  display: inline-flex;
  align-items: center;
  gap: var(--space-2);
  padding: var(--space-2) var(--space-4);
  border-radius: var(--radius-sm);
  font-family: inherit;
  font-size: 13.5px;
  font-weight: 500;
  cursor: pointer;
  border: none;
  transition: background 0.15s;
}

.btn-primary {
  background: var(--accent);
  color: white;
}
.btn-primary:hover { background: var(--accent-hover); }

.btn-ghost {
  background: transparent;
  color: var(--text-primary);
  border: 1px solid var(--border);
}
.btn-ghost:hover { background: var(--bg-hover); }

/* Icon-only button */
.icon-btn {
  background: transparent;
  border: none;
  cursor: pointer;
  padding: var(--space-2);
  border-radius: var(--radius-sm);
  color: var(--text-secondary);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.15s, color 0.15s;
}
.icon-btn:hover {
  background: var(--bg-hover);
  color: var(--text-primary);
}
```

### Card
```css
.card {
  background: var(--bg-surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  padding: var(--space-6);
  box-shadow: var(--shadow-sm);
}
```

### Input / Text Field
```css
.input {
  width: 100%;
  padding: var(--space-2) var(--space-3);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  background: var(--bg-surface);
  color: var(--text-primary);
  font-family: inherit;
  font-size: 14px;
}
.input::placeholder { color: var(--text-muted); }
.input:focus {
  outline: none;
  border-color: var(--accent);
}
```

### Table
```css
.table {
  width: 100%;
  border-collapse: collapse;
  font-size: 13.5px;
}
.table th {
  text-align: left;
  padding: var(--space-2) var(--space-3);
  border-bottom: 2px solid var(--border-strong);
  color: var(--text-secondary);
  font-weight: 600;
}
.table td {
  padding: var(--space-2) var(--space-3);
  border-bottom: 1px solid var(--border);
}
.table tr:hover td {
  background: var(--bg-hover);
}
```

---

## Dark Mode Logic (`app.js`)

Always include this at the top of `app.js`:

```js
// === Dark mode toggle ===
// Cmd+Shift+L (Mac) or Ctrl+Shift+L (Windows) — or click the button
const themeToggle = document.getElementById('themeToggle');
const themeIcon = document.getElementById('themeIcon');

function setTheme(dark) {
  document.documentElement.setAttribute('data-theme', dark ? 'dark' : 'light');
  // Swap icon
  if (themeIcon) {
    themeIcon.setAttribute('data-lucide', dark ? 'sun' : 'moon');
    lucide.createIcons(); // re-render after attribute change
  }
  localStorage.setItem('theme', dark ? 'dark' : 'light');
}

// On load: restore saved preference, fall back to OS setting
const saved = localStorage.getItem('theme');
const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
setTheme(saved === 'dark' || (!saved && prefersDark));

// Button
if (themeToggle) {
  themeToggle.addEventListener('click', () => {
    setTheme(document.documentElement.getAttribute('data-theme') !== 'dark');
  });
}

// Keyboard shortcut
document.addEventListener('keydown', (e) => {
  if ((e.metaKey || e.ctrlKey) && e.shiftKey && e.key === 'L') {
    setTheme(document.documentElement.getAttribute('data-theme') !== 'dark');
  }
});
```

---

## Full `index.html` Shell

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>App Name</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
  <script src="https://unpkg.com/lucide@latest/dist/umd/lucide.min.js"></script>
  <link rel="stylesheet" href="css/style.css">
</head>
<body>

  <!-- layout goes here -->

  <script src="js/app.js"></script>
  <script>lucide.createIcons();</script>
</body>
</html>
```

---

## Notes for Claude

- Rockson comes from Flutter/Dart — use Flutter analogies when explaining web concepts:
  - HTML = Widget tree, `<div>` = `Container`
  - CSS variables = `ThemeData` tokens
  - Flexbox = `Row` / `Column`
  - JS functions = Dart methods
- Never use React, Vue, npm, or build tools. Pure HTML/CSS/JS only.
- Firebase is a separate skill — don't include it here unless asked.
- After generating files, give one sentence per file explaining what it does.
- Keep code readable over clever. Rockson reads code to understand it.
