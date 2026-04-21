---
name: design-directions
description: "Generate hi-fi HTML prototype variants from context files, arranged on a tabbed canvas with live Tweaks controls. Mirrors the Claude Design prototype flow. Use when Dan says '/design-directions', 'design directions', or provides context files and asks for multiple visual directions."
argument-hint: <project name or brief>
---

# Prototype Skill — Multi-Variant HTML Prototyping

Replicates the Claude Design prototype outcome: ingest context → clarifying questions → multiple standalone hi-fi HTML variants + a canvas file with tabs, sticky notes, and a live Tweaks sidebar. Output lives in `/Users/dancarino/Vibe Projects/<project-slug>/`.

---

## Phase 1 — Load context

1. **Identify the project.** `$ARGUMENTS` is the project name or brief. If empty, ask Dan for a name.
2. **Collect context.** Read any files Dan attached or referenced. If Dan points at a folder, read its index file + any prompt/spec files inside. If nothing is attached, ask what to use.
3. **Acknowledge.** One short paragraph: "I read X, Y, Z. Here's my understanding of the goal: …" — then proceed to questions.

---

## Phase 2 — Clarifying questions (ALWAYS ask — never skip)

Ask in one batch with a default on each:

1. **Scope** — one variant or multiple on a canvas? (Default: 4 variants.)
2. **Fidelity** — wireframe, hi-fi static, or interactive? (Default: hi-fi static.)
3. **Key screen** — which single screen is THE screen? (Default: ask Dan — don't guess.)
4. **Differentiation axis** — what should variants differ on? (Default: typography + layout + tone, same content.)
5. **Extras** — any must-have UI pattern? (Default: none.)

Wait for answers. Do not infer silently.

---

## Phase 3 — Plan

Name each variant with a one-line visual concept + the shared content. Confirm before building.

---

## Phase 4 — Generate

### 4a. Variant HTML files

Each variant is `variant-<letter>-<slug>.html`. Rules:
- Single file — inline `<style>`, inline `<script>` if needed, no build step, no JS frameworks.
- Google Fonts via `<link>` allowed. No other external assets.
- Same content across all variants — only visual treatment changes.
- Real content pulled from context files, never lorem ipsum.
- Small header comment naming the variant and its concept.

**CSS variable architecture (required on every variant):**
Every variant must define these four CSS custom properties on `:root` and use them in the relevant selectors:
```css
:root {
  --accent: #hexcolor;   /* primary accent / brand color */
  --bg: #hexcolor;       /* page background */
  --h1-size: Npx;        /* hero headline font-size */
  --body-size: Npx;      /* main body / textarea / description font-size */
}
```
Wire each to its selector (e.g. `font-size: var(--h1-size)`, `background: var(--bg)`). Add variant-specific extras (e.g. `--btn-radius`, `--mono-size`) as needed.

**postMessage listener (required on every variant):**
Add this at the end of `<body>` so the canvas can push CSS var changes live:
```html
<script>
  window.addEventListener('message', e => {
    if (e.data && e.data.type === 'set-var')
      document.documentElement.style.setProperty(e.data.prop, e.data.value);
  });
</script>
```

### 4b. Canvas file

Filename: `<project-slug>-canvas.html`. This is the main file Dan opens.

**Canvas chrome — visual spec (labs.google light mode aesthetic):**
- Font: `Plus Jakarta Sans` (via Google Fonts, weights 400/500/600/700)
- Background: `#f8f9fa`
- Body text: `#202124`

**Structure:**

1. **Top nav bar** (fixed, 56px, `background: rgba(248,249,250,0.92)`, `backdrop-filter: blur(20px)`, `border-bottom: 1px solid rgba(0,0,0,0.07)`):
   - Left: artist palette SVG icon (with Google-colored dots: `#ea4335`, `#fbbc04`, `#34a853`, `#4285f4` and a blue→purple gradient stroke) + project title (bold, `#202124`) + 1px separator + **tab bar**
   - Tab pills: `border-radius: 20px`; default color `#80868b`; active state `background: #202124; color: #fff; font-weight: 600`
   - Right: **Tweaks** button — `background: linear-gradient(135deg, #4285f4 0%, #9b57e0 100%)`, white text, `border-radius: 20px`; active state switches to green→blue gradient

2. **Canvas tab** (default view): 2-column grid of variant cards. Each card has:
   - A label row (background `#fafafa`, border-bottom `rgba(0,0,0,0.05)`): variant name (bold), one-line concept (muted), pill-shaped "Open ↗" link
   - The iframe (aspect-ratio container, `padding-top: 75%`, background `#f8f9fa`)
   - A **sticky note** below the card — written by Claude, not by Dan. Alternating left/right rotation per row.

   **Sticky note spec:**
   - Font: `Patrick Hand` (via Google Fonts) at `17px`, `line-height: 1.45`
   - Size: `width: 272px; aspect-ratio: 16/9; overflow: hidden` — text should nearly fill the note
   - Color: `background: #fff475; color: #3c3c00`
   - `transform: rotate(-1.2deg)` for odd rows, `rotate(1.4deg)` for even rows (`.variant:nth-child(even)`)
   - `box-shadow: 0 4px 16px -4px rgba(100,80,0,0.2), 0 1px 3px rgba(0,0,0,0.08)`
   - Top gradient fade: `::before` with `height: 8px; background: linear-gradient(180deg, rgba(0,0,0,0.05), transparent)`
   - Keep the tradeoff text tight — one or two sentences max so it fills the 16:9 frame

   **Card spec:**
   - `background: #fff; border-radius: 16px; border: 1px solid rgba(0,0,0,0.07)`
   - `box-shadow: 0 1px 3px rgba(0,0,0,0.04), 0 12px 32px -8px rgba(0,0,0,0.08)`
   - Hover: deeper shadow + `border-color: rgba(0,0,0,0.1)`

3. **Variant tabs (A/B/C/D)**: clicking a variant tab switches to full-screen mode:
   - `body.variant-mode` class added; the active variant's card becomes `position: fixed; top: 56px; inset: 0` filling the screen
   - Other variants hidden. Sticky notes hidden in variant mode.
   - The iframe fills the fixed card (`width: 100%; height: 100%`)
   - Tweaks sidebar auto-switches to that variant's controls

4. **Right sidebar** (slides in from right, 340px wide, pushes the body — `padding-right: calc(340px + 40px)` on `body.sidebar-open`):
   - `background: #fff; border-left: 1px solid rgba(0,0,0,0.07)`
   - Variant switcher pills: default `background: #f1f3f4`; active state uses the blue→purple gradient
   - `accent-color: #4285f4` on range inputs
   - Controls rendered from a `TWEAKS` JS object, grouped into **Typography**, **Colors**, **Specific** sections
   - Hint text at the bottom in `#bdc1c6`
   - In variant mode: `right: 340px` on the fixed card (not `inset: 0`) when sidebar is open

**TWEAKS JS object** (pre-populated for all variants):
```js
const TWEAKS = {
  a: {
    name: 'A · [Name]',
    iframe: 0,
    controls: [
      { id: 'h1',     label: 'Headline',   prop: '--h1-size',   type: 'range', min: 28, max: 80, value: N,  unit: 'px', section: 'Typography' },
      { id: 'body',   label: 'Body',       prop: '--body-size', type: 'range', min: 13, max: 24, value: N,  unit: 'px', section: 'Typography' },
      { id: 'accent', label: 'Accent',     prop: '--accent',    type: 'color',                  value: '#hex',          section: 'Colors' },
      { id: 'bg',     label: 'Background', prop: '--bg',        type: 'color',                  value: '#hex',          section: 'Colors' },
      // variant-specific extras in section: 'Specific'
    ]
  },
  // b, c, d follow same shape
};
```

**Control rendering:**
- `type: 'range'` → `<input type="range">` with label + live value readout
- `type: 'color'` → `<input type="color">` styled as a small swatch with hex label beside it
- Each section gets a small uppercase section header

**Wiring (always use postMessage — not direct DOM access):**
```js
iframes[t.iframe].contentWindow.postMessage({ type: 'set-var', prop: c.prop, value: v }, '*');
```

**Tab switching JS:**
```js
tabsEl.addEventListener('click', e => {
  const btn = e.target.closest('button[data-tab]');
  if (!btn) return;
  activeTab = btn.dataset.tab;
  // update tab button active states
  if (activeTab === 'canvas') {
    document.body.classList.remove('variant-mode');
    document.querySelectorAll('.variant').forEach(v => v.classList.remove('tab-active'));
  } else {
    document.body.classList.add('variant-mode');
    document.querySelectorAll('.variant').forEach(v => v.classList.remove('tab-active'));
    document.querySelector(`.variant[data-key="${activeTab}"]`).classList.add('tab-active');
    // sync Tweaks sidebar
    active = activeTab;
    // update variant-switch pills + re-render controls
  }
});
```

### 4c. README.md

Brief, variant names + concepts, and: `open "<slug>-canvas.html"`.

---

## Phase 5 — Deliver

- Path to the folder
- One line per variant
- `open` command for the canvas
- Reminder that Tweaks are pre-populated and he can ask for more controls, bake-in, or direct edits

---

## Phase 6 — Iterate (stay open after delivery)

### (a) Direct tweaks
Dan says "make the accent greener on B" → edit the variant file directly, no controls added.

### (b) Live controls (pre-populated by default, add more on request)
Every canvas ships with the 4 shared controls pre-wired (Typography + Colors). When Dan asks for more — e.g. "add a border-radius slider to C" — add a new entry to the variant's `controls` array in the `TWEAKS` object, and add the corresponding CSS var to the variant HTML if it doesn't exist yet.

### (c) Bake-in and remove
When Dan says "keep that value" → hard-code it in the variant's base CSS and remove the control entry from `TWEAKS`. The controls are scaffolding — they don't live in the final file.

---

## Guardrails

- **Never skip Phase 2 questions.**
- **No frameworks.** Vanilla HTML/CSS/JS only — Dan edits these by hand.
- **Real content only.** Pull from context files. If context is thin, ask.
- **Vibe Projects is outside the vault.** No `[[wikilinks]]` in generated files.
- **One project = one folder.**
- **Always use postMessage for iframe wiring**, never `contentDocument` direct access (breaks on `file://`).
- **Every variant must define the 4 standard CSS vars** (`--accent`, `--bg`, `--h1-size`, `--body-size`) and include the postMessage listener — this is the contract the canvas depends on.
