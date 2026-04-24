---
name: design-directions
description: "Generate hi-fi HTML prototype variants from context files, arranged on a tabbed canvas with live Tweaks controls. Mirrors the Claude Design prototype flow. Use when the user says '/design-directions', 'design directions', or provides context files and asks for multiple visual directions."
argument-hint: <project name or brief>
---

# Prototype Skill — Multi-Variant HTML Prototyping

Replicates the Claude Design prototype outcome: ingest context → clarifying questions → multiple standalone hi-fi HTML variants + a canvas file with tabs, sticky notes, and a live Tweaks sidebar. Output lives in a **new subfolder** inside `~/design-directions/` by default — honor a different location if the user specifies one. Never write to the root and never mix output into an existing project folder.

---

## Phase 1 — Load context

1. **Identify the project.** `$ARGUMENTS` is the project name or brief. If empty, ask the user for a name.
2. **Collect context.** Read any files the user attached or referenced. If the user points at a folder, read its index file + any prompt/spec files inside. If nothing is attached, ask what to use.
3. **Acknowledge.** One short paragraph: "I read X, Y, Z. Here's my understanding of the goal: …" — then proceed to questions.

---

## Phase 2 — Clarifying questions (ALWAYS ask — never skip)

Ask in one batch with a default on each:

1. **Platform** — website or app? If app: iOS, Android, or cross-platform? (Default: website. If the project name or brief mentions "app", "mobile", "iOS", or "Android", default to app.)
2. **Scope** — one variant or multiple on a canvas? (Default: 4 variants.)
3. **Fidelity** — wireframe, hi-fi static, or interactive? (Default: hi-fi static.)
4. **Key screen** — which single screen is THE screen? (Default: ask the user — don't guess.)
5. **Differentiation axis** — what should variants differ on? (Default: typography + layout + tone, same content.)
6. **Extras** — any must-have UI pattern? (Default: none.)
7. **Visual reference** — present this as a choice using AskUserQuestion with the following options:

   - **Linear** — dark-first, indigo accent, Inter weight 510, precision engineering feel
   - **Stripe** — light, deep navy + purple, weight-300 elegance, blue-tinted shadows
   - **Notion** — warm neutrals, whisper borders, approachable minimalism
   - **Vercel** — pure monochrome, Geist font, shadow-as-border, gallery emptiness
   - **Figma** — black + white only, variable font, pill/circle geometry, vibrant product colour
   - **Spotify** — dark immersive, green accent, pill buttons, heavy shadows
   - **Custom** — I'll describe a style or paste a URL
   - **None** — derive tone from context files only

   If the user picks Custom, ask for the style description or URL. If a URL is given, fetch it. If a brand name is given, run `npx getdesign@latest add <brand>` in a temp directory and read the generated DESIGN.md. If that command fails, ask the user to paste a URL or describe the style in a few sentences.

8. **Canvas** — generate a canvas file with tabs, sticky notes, and a live Tweaks sidebar? (Default: Yes.) Present as an AskUserQuestion choice:
   - **Yes — canvas + variants** *(default)* — canvas file wrapping all variants with tabs, sticky notes, and Tweaks sidebar
   - **No — variants only** — just the standalone HTML files, no canvas wrapper

Wait for answers. Do not infer silently.

---

## Phase 3 — Plan

Name each variant with a one-line visual concept + the shared content. Confirm before building.

---

## Phase 4 — Generate

### Optional: extra design reference files

If the user has personal design-reference markdown files (a notes vault, a design docs folder, etc.), read them for extra guidance. If they don't exist, skip this step — the skill works without them.

**How to find them:** if the user explicitly pointed at a folder, check there. Otherwise use Glob to probe these common locations (silently skip any that are missing — don't report them):

- `~/design-notes/`
- `~/Design/`
- `~/notes/design/`
- Any folder in the current project matching `**/design-notes/` or `**/design/reference/`

**What to look for** — markdown files covering:

*Always useful:*
- Typography and font pairing
- Colour palette construction
- Brand tone → visual language mapping
- UI component patterns (hero, card, CTA, nav, form, empty state)
- Motion and easing reference
- UX writing and UI copy

*For app platform — also:* mobile app design principles, current mobile design trends

*For website platform — also:* web design principles, current web design trends

Match by filename keywords (e.g. "typography", "colour"/"color", "motion", "mobile"). If the user has a vault structure you can detect, read those files. If nothing matches, proceed without them — platform-specific requirements below still apply.

### Platform-specific requirements (apply regardless of reference files)

For **app** variants: thumb zone layout, bottom-anchored nav, gesture patterns, and 44pt tap targets are required — not optional. For **website** variants: hover states, responsive breakpoints, and clear visual hierarchy are required.

### Visual reference handling

If a **visual reference** was chosen in Phase 2 Q7:

- **Named brand (Linear / Stripe / Notion / Vercel / Figma / Spotify or any other):** run `npx getdesign@latest add <brand>` in a temp directory and read the generated DESIGN.md. `getdesign` is a published npm package ([getdesign.md](https://getdesign.md)) — if the command fails (network error, unknown brand, npx unavailable), fall back by asking the user to either paste a URL for that brand's site or describe the style in a few sentences.
- **Custom URL:** fetch it directly.
- **Custom description:** use the description as the base aesthetic layer.

Extract: tone, font, colour palette, shape language (border radius), shadow system, spacing philosophy, motion feel, and Do's/Don'ts. Apply these as the base aesthetic layer before differentiating variants. If the DESIGN.md includes an "Agent Prompt Guide" section, treat it as the literal spec for component generation.

### 4a. Variant HTML files

Each variant is `variant-<letter>-<slug>.html`. Rules:
- Single file — inline `<style>`, inline `<script>` if needed, no build step, no JS frameworks.
- Google Fonts via `<link>` allowed. No other external assets.
- Same content across all variants — only visual treatment changes.
- Real content pulled from context files, never lorem ipsum.
- Small header comment naming the variant and its concept.

**Platform mode — apply based on Phase 2 answer:**

*Website (default):* Standard full-width layout. No special frame.

*App:* Wrap the screen in a phone shell so it previews at realistic mobile dimensions inside the iframe:
```html
<body style="margin:0; background:#e8e8ed; display:flex; align-items:center; justify-content:center; min-height:100vh;">
  <div class="phone-frame">
    <!-- status bar -->
    <div class="status-bar">
      <span>9:41</span>
      <span style="margin-left:auto">···</span>
    </div>
    <!-- scrollable app content -->
    <div class="app-content">
      <!-- screen content here -->
    </div>
    <!-- home indicator -->
    <div class="home-indicator"></div>
  </div>
</body>
```
Phone frame CSS (add to `<style>`):
```css
.phone-frame {
  width: 390px; height: 844px;
  background: var(--bg);
  border-radius: 48px;
  box-shadow: 0 0 0 12px #1c1c1e, 0 32px 64px rgba(0,0,0,0.4);
  overflow: hidden;
  display: flex; flex-direction: column;
  position: relative;
}
.status-bar {
  height: 44px; padding: 14px 24px 0;
  font-size: 15px; font-weight: 600;
  color: var(--text-primary, #000);
  display: flex; align-items: center;
  flex-shrink: 0;
}
.app-content {
  flex: 1; overflow-y: auto;
  padding-bottom: env(safe-area-inset-bottom, 34px);
  -webkit-overflow-scrolling: touch;
}
.home-indicator {
  height: 34px; display: flex;
  align-items: center; justify-content: center; flex-shrink: 0;
}
.home-indicator::after {
  content: ''; width: 134px; height: 5px;
  background: var(--text-primary, #000); opacity: 0.2;
  border-radius: 3px;
}
```
For app variants, also add `--text-primary` to the CSS variable architecture and set `<meta name="viewport" content="width=390">`.

**CSS variable architecture (required on every variant):**
Every variant must define these CSS custom properties on `:root` and use them in the relevant selectors:
```css
:root {
  --accent: #hexcolor;         /* primary accent / brand color */
  --bg: #hexcolor;             /* page / app background */
  --h1-size: Npx;              /* hero / screen title font-size */
  --body-size: Npx;            /* main body / description font-size */
  /* App variants only: */
  --text-primary: #hexcolor;   /* primary text (status bar, labels) */
}
```
Wire each to its selector (e.g. `font-size: var(--h1-size)`, `background: var(--bg)`). Add variant-specific extras (e.g. `--btn-radius`, `--tab-bar-bg`) as needed.

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

**Skip this section entirely if the user answered "No — variants only" in Phase 2 Q8.** Jump straight to 4c.

Filename: `<project-slug>-canvas.html`. This is the main file the user opens.

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
   - The iframe — use aspect-ratio based on platform:
     - *Website:* `padding-top: 75%` (4:3), background `#f8f9fa`
     - *App:* `padding-top: 216%` (9:19.5 to match phone shell height at 390px width), background `#e8e8ed`. **Do NOT add a phone-frame wrapper div around the iframe** — the variant HTML already renders its own phone shell. Adding an outer frame creates a double-frame. The canvas container is just a clipping rectangle; use `border-radius: 35px; overflow: hidden; box-shadow: 0 8px 32px rgba(0,0,0,0.22)` for shape and depth without a dark border.
   - A **sticky note** below the card — written by Claude, not by the user. Alternating left/right rotation per row.

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
   - Other variants hidden with `display: none` applied **directly to `.variant` elements** — never to their parent container (`.variants-grid` or `#canvas-area`). A `display: none` or `visibility: hidden` on a parent hides `position: fixed` descendants too, which breaks the full-screen tab. The parent grid collapses naturally when all its direct children are `display: none`.
   - Sticky notes hidden in variant mode.
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

*Website controls:*
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

*App controls (replace the range maxes and add app-specific extras):*
```js
const TWEAKS = {
  a: {
    name: 'A · [Name]',
    iframe: 0,
    controls: [
      { id: 'h1',     label: 'Screen title', prop: '--h1-size',        type: 'range', min: 17, max: 34, value: N,  unit: 'px', section: 'Typography' },
      { id: 'body',   label: 'Body',         prop: '--body-size',      type: 'range', min: 13, max: 20, value: N,  unit: 'px', section: 'Typography' },
      { id: 'accent', label: 'Accent',       prop: '--accent',         type: 'color',                  value: '#hex',          section: 'Colors' },
      { id: 'bg',     label: 'Background',   prop: '--bg',             type: 'color',                  value: '#hex',          section: 'Colors' },
      { id: 'text',   label: 'Text',         prop: '--text-primary',   type: 'color',                  value: '#hex',          section: 'Colors' },
      // variant-specific extras (e.g. --tab-bar-bg, --btn-radius) in section: 'Specific'
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

Brief, variant names + concepts.
- If canvas was generated: include `open "<slug>-canvas.html"` as the launch command.
- If variants only: list each file with `open "<slug>/variant-<letter>-<slug>.html"` for individual viewing.

---

## Phase 5 — Deliver

- Path to the folder
- One line per variant
- If canvas was generated: `open` command for the canvas + reminder that Tweaks are pre-populated and the user can ask for more controls, bake-in, or direct edits
- If variants only: `open` command for each standalone variant file

---

## Phase 6 — Iterate (stay open after delivery)

### (a) Direct tweaks
The user says "make the accent greener on B" → edit the variant file directly, no controls added.

### (b) Live controls (pre-populated by default, add more on request)
Every canvas ships with the 4 shared controls pre-wired (Typography + Colors). When the user asks for more — e.g. "add a border-radius slider to C" — add a new entry to the variant's `controls` array in the `TWEAKS` object, and add the corresponding CSS var to the variant HTML if it doesn't exist yet.

### (c) Bake-in and remove
When the user says "keep that value" → hard-code it in the variant's base CSS and remove the control entry from `TWEAKS`. The controls are scaffolding — they don't live in the final file.

---

## Guardrails

- **Never skip Phase 2 questions.**
- **No frameworks.** Vanilla HTML/CSS/JS only — the user edits these by hand.
- **Real content only.** Pull from context files. If context is thin, ask.
- **No `[[wikilinks]]` or notes-app-specific syntax** in generated files — output is plain HTML.
- **One project = one folder.**
- **Always create a new subfolder before writing any files.** Before generating, check whether `~/design-directions/<project-slug>/` already exists and contains files. If it does, create a new subfolder inside it (e.g. `round-2/`, a descriptive slug, or a date suffix) and generate everything there. Never write variant files into a folder that already has HTML files.
- **Always use postMessage for iframe wiring**, never `contentDocument` direct access (breaks on `file://`).
- **Every variant must define the 4 standard CSS vars** (`--accent`, `--bg`, `--h1-size`, `--body-size`) and include the postMessage listener — this is the contract the canvas depends on.
- **Never hide a parent of a `position: fixed` child using `display: none` or `visibility: hidden`.** Both properties cascade to fixed descendants and break the full-screen tab mode. Hide non-active variants directly (`body.variant-mode .variant { display: none }`) and let the empty parent grid collapse on its own.
