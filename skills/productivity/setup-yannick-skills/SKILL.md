---
name: setup-yannick-skills
description: "Configure where the skills park decisions for the user (GitHub/GitLab issues, local markdown, or any ticket system) and how they get work independently reviewed — per repo or for all projects. Run once before the first /autonomous run."
disable-model-invocation: true
---

# Setup Yannick's Skills

Scaffold the configuration the skills read when the user is not around to ask:

- **Decision sink**: where decisions that only the user can make are filed, so they outlive the session and the
  user can answer them on their own schedule
- **Review**: how work gets an independent review before it is committed, in place of the user looking over it

`/autonomous` is the main consumer: it cannot ask mid-run where to put a decision, because the user is away —
so the answer has to be settled here, beforehand.

This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the
user, then write.

## Process

### 1. Explore

Read whatever exists; don't assume:

- `git remote -v`: is there a repo at all? GitHub, GitLab, or something else?
- `CLAUDE.md` and `AGENTS.md` at the repo root, and `~/.claude/CLAUDE.md`: is there already an `## Agent skills`
  block in any of them?
- `docs/agents/` and `~/.claude/skills-config/`: does this skill's prior output already exist?
- `gh` / `glab` on the PATH and authenticated?
- Your available skills and MCP tools: is there one that already files tickets or decisions (a ticket-system
  skill, a Linear/Jira connector)? If so, it is a strong candidate for the decision sink.

**Done when:** you can name the current state for scope, decision sink and review, including any prior config.

### 2. Present findings and ask

Summarise what's present and what's missing. Then take the sections in order — one section, one answer, then
the next. Lead each with the recommended answer so the user can accept it in a word; skip a section when
exploration already settled it.

**Section A: Scope.**

> Should this apply to this repo only, or to all your projects?

- **This repo** (recommended inside a repo): files under `docs/agents/`, block in the repo's `CLAUDE.md`/`AGENTS.md`.
- **All projects**: files under `~/.claude/skills-config/`, block in `~/.claude/CLAUDE.md`. Right for a ticket
  system the user runs across all projects.

Repo config overrides user config, so a user-wide default can be replaced in single repos later.

**Section B: Decision sink.**

> Explainer: when a skill hits a decision only you can make, it parks it with a provisional assumption and keeps
> working. The decision sink is where that question is filed so you can answer it later — pick the place you
> actually look.

Propose based on exploration: a ticket skill or connector the user already uses → that; a GitHub remote →
GitHub; a GitLab remote → GitLab; otherwise local markdown.

For GitHub/GitLab, settle the target too: with repo scope, pin `owner/name` from the remote; with user scope,
ask which owners/namespaces (their own account, their orgs) may receive decisions — any other repo falls back
to the parking lot. If `gh`/`glab` isn't authenticated, have the user run `gh auth login` / `glab auth login`
now; an unattended run can't. Options:

- **GitHub**: issues labelled `needs-decision` (uses `gh`) — seed: [decision-sink-github.md](./decision-sink-github.md)
- **GitLab**: issues labelled `needs-decision` (uses `glab`) — seed: [decision-sink-gitlab.md](./decision-sink-gitlab.md)
- **Local markdown**: one file per decision under `docs/decisions/` — seed: [decision-sink-local.md](./decision-sink-local.md)
- **Other** (a ticket system, Linear, Jira, a team chat, a skill or MCP tool): ask the user to describe in one
  paragraph how to file a decision and how to see whether it was answered; record that as freeform prose in the
  same shape as the seeds (target / file / check for answers / link).

**Section C: Review.**

> Explainer: `/autonomous` must get every block independently reviewed before committing, because nobody is
> reading along. How should that review run?

- **Fresh subagent** (recommended default, works everywhere): a reviewer subagent with no context from the
  implementation, given the diff and the block's goal.
- **Second model / CLI**: the user names the command (e.g. another vendor's code-review CLI); record the exact
  invocation.
- **Both**: second model first, subagent cross-check.

Seed: [review.md](./review.md).

**Done when:** scope, decision sink and review each have an answer the user confirmed.

### 3. Confirm and write

Show the user a draft of the `## Agent skills` block and of both config files. Let them edit before writing.

**Pick the file for the block:**

- Scope *all projects*: `~/.claude/CLAUDE.md` (create it if missing).
- Scope *this repo*: `CLAUDE.md` if it exists, else `AGENTS.md` if it exists; if neither exists, ask the user
  which one to create — don't pick for them. Never create one when the other already exists.

If an `## Agent skills` block already exists in that file, update its contents in place rather than appending a
duplicate. Don't touch the surrounding sections.

The block, with `<config dir>` = `docs/agents` (repo scope, relative to the repo root) or the **expanded
absolute path** of `~/.claude/skills-config` (user scope, e.g. `/home/alex/.claude/skills-config` — never
`docs/agents` and never `~`, or a run in another repo won't find it):

```markdown
## Agent skills

### Decision sink

[one line: where decisions go]. See `<config dir>/decision-sink.md`.

### Review

[one line: how independent review runs]. See `<config dir>/review.md`.
```

Then write `decision-sink.md` and `review.md` into `<config dir>`, using the seeds in this folder as a starting
point: keep only the Target bullet for the chosen scope (delete the other) and fill in its placeholders
(target repo or allowed owners). With repo scope on GitHub/GitLab, offer
to create the `needs-decision` label now (it's visible to collaborators, so ask); otherwise the first filing
creates it, as the seed allows.

**Done when:** the block exists exactly once in the chosen file, its paths resolve from any working directory
in the chosen scope, both config files exist with no placeholders left and say where to file, how to file,
check and link a decision, and the user has seen the final text.

### 4. Done

Tell the user the setup is complete, where it lives, and which skills read it (`/autonomous` for both sections).
They can edit the files directly later; re-running this skill is only needed to switch sinks or scope.
