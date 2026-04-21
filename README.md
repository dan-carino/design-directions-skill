# design-directions — Claude Code Skill

A Claude Code skill that generates multiple hi-fi HTML prototype variants from your context files, arranged on a tabbed canvas with live Tweaks controls. Mirrors the Claude Design prototype flow.

## What it does

Run `/design-directions` with a project name or brief and it will:

1. Read your context files (Figma notes, briefs, spec docs)
2. Ask 5 clarifying questions about scope, fidelity, and differentiation
3. Generate 4 standalone HTML variants — each a different visual direction for the same content
4. Wrap them in a canvas file with tabs, sticky notes, and a live Tweaks sidebar (accent color, background, font sizes)
5. Stay open for iteration — direct edits, new controls, or baking in values

No frameworks. No build step. Pure HTML/CSS/JS you can open in a browser and edit by hand.

## Output

```
/Vibe Projects/<project-slug>/
  variant-a-<slug>.html
  variant-b-<slug>.html
  variant-c-<slug>.html
  variant-d-<slug>.html
  <project-slug>-canvas.html   ← open this
  README.md
```

## Install

### Option A — copy to personal skills (available in all your projects)

```bash
cp -r .claude/skills/design-directions ~/.claude/skills/
```

### Option B — copy to a specific project

```bash
cp -r .claude/skills/design-directions /path/to/your/project/.claude/skills/
```

Then in Claude Code, run:

```
/design-directions <your project name or brief>
```

## Requirements

- [Claude Code](https://claude.ai/code)
- The skill outputs to `/Vibe Projects/<project-slug>/` by default — edit `SKILL.md` line 9 to change the output path

## Notes

- The canvas uses `file://` iframes with `postMessage` for live Tweaks — no server needed
- Each variant defines 4 standard CSS vars (`--accent`, `--bg`, `--h1-size`, `--body-size`) that the Tweaks sidebar controls
- Real content only — the skill pulls from your context files, never lorem ipsum
