# Skills repo — conventions

Public collection of my agent skills, installable as a Claude Code plugin and via `npx skills`.

## Layout

- `skills/<category>/<skill-name>/SKILL.md` — one folder per skill. Categories: `engineering`, `productivity`.
  Supporting files (reference docs, scripts) live next to `SKILL.md` in the same folder.
- `.claude-plugin/plugin.json` — lists every published skill under `skills`.
- `.claude-plugin/marketplace.json` — makes the repo installable as a marketplace.

## Adding a skill

1. Create `skills/<category>/<name>/SKILL.md` with frontmatter: `name` (matches the folder), `description`
   (what it does + when to use it + `Usage:` line), optional `argument-hint`. Add
   `disable-model-invocation: true` for skills the agent must never start on its own.
2. Write in English. Keep it project-agnostic: no references to private projects, internal tools or names.
3. Structure the body as phases, each ending in a **Done when:** criterion that can be checked.
4. Register the path in `.claude-plugin/plugin.json` and add a row + section to `README.md`.
5. Bump `version` in `plugin.json` (minor for a new skill, patch for fixes).
6. Validate: `claude plugin validate .`
