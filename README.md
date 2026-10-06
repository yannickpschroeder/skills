# Skills

Agent skills I built and use in my own projects — for work that has to hold up when nobody is watching.

Coding agents are good at producing changes. They are much worse at knowing when a change is *actually* done,
and at keeping going without a human in the loop. These skills address exactly that: every step ends in a
**checkable done criterion**, failures are pinned as **red tests before anything is fixed**, and questions that
don't truly block the work are **parked instead of asked**.

They work with Claude Code and any other agent that reads `SKILL.md` files.

## Skills

| Skill | What it does |
| --- | --- |
| [`/harden`](./skills/engineering/harden/SKILL.md) | Hardens a system in a red-green loop: probe for weaknesses, pin them as failing cases, audit the test bench itself, fix until green, report. |
| [`/autonomous`](./skills/productivity/autonomous/SKILL.md) | Runs for hours while you're away: plans blocks with done criteria, works through them without asking, parks questions, stops only on true blockers. |
| [`/setup-yannick-skills`](./skills/productivity/setup-yannick-skills/SKILL.md) | One-time setup: where parked decisions are filed (GitHub/GitLab issues, local files, any ticket system) and how work gets independently reviewed — per repo or for all projects. |

### `/harden` — red-green hardening loop

```
/harden the invoice parser
```

Most "make it more robust" requests end in vague refactors. `/harden` turns robustness into a measurable loop:

1. **Probe** — read the existing test bench case by case, then hunt for weaknesses it doesn't show yet. Every
   weakness needs a concrete scenario (input → wrong behaviour); "could be more robust" doesn't count.
2. **Pin red** — one new case per weakness, built from real data where possible. Every new case must fail
   first — a case that passes immediately proves nothing.
3. **Bench audit** — before fixing anything, check that the bench measures correctly: mutation spot-checks for
   blind spots, independent oracle checks for false alarms, matcher sanity, repeat runs for non-deterministic
   systems (e.g. LLM outputs), and pruning of truly redundant cases.
4. **Green** — change only the system, never weaken the test, no overfitting to pinned inputs.
5. **Report** — ledger of weakness → case → before/after, then ask whether to run another round. Fixes move the
   weakest spot, so each round starts with fresh probing.

### `/autonomous` — long unattended runs

```
/autonomous docs/plan.md 6h
```

When you leave an agent alone for hours, it usually either stops at the first question or improvises past
things it shouldn't touch. `/autonomous` sets clear rules:

- **True blockers only.** The run stops only if the next step is irreversible or crosses a guardrail *and*
  there's no defensible provisional assumption *and* no other block can run instead. Everything else goes to a
  **parking lot** with the assumption that was made.
- **Plan in blocks.** Each block has a checkable done criterion, dependencies and an estimate, plus a buffer so
  the run doesn't idle after two hours.
- **Review replaces the human.** Every block gets an independent review before it's committed — mandatory,
  precisely because nobody is watching.
- **Survives context compaction.** A progress log is written after every block; after a compaction the run
  resumes from plan and log.
- **Hand-off.** On return you get a report and a list of decisions to make — no need to read the transcript.

`/autonomous` is user-invoked only (`disable-model-invocation`), so the agent never starts an unattended run
on its own.

### `/setup-yannick-skills` — configure once, before you leave

An unattended run can't ask you where to put a decision — so that's settled up front. The setup asks four
questions and writes the answers to an `## Agent skills` block in your `CLAUDE.md`/`AGENTS.md`:

- **Scope** — this repo only, or all your projects (`~/.claude/CLAUDE.md`). Repo config overrides user config.
- **Decision sink** — GitHub or GitLab issues labelled `needs-decision`, one markdown file per decision, or
  any other system you describe in a sentence (a ticket system, Linear, Jira, a skill or MCP tool you already use).
- **Language** — which language decisions and reports are written in, independent of the skills' own language.
- **Review** — a fresh reviewer subagent, a second model via its CLI, or both.

`/autonomous` then files each decision the moment it parks it, picks up your answers at the start of the next
run, and reviews every block the configured way. Without setup it still works and keeps decisions in a local
parking-lot file.

## Installation

### Claude Code plugin

```bash
claude plugin marketplace add yannickpschroeder/skills
claude plugin install yannickschroeder-skills@yannickpschroeder
```

### Any agent (copies editable files into your project)

```bash
npx skills@latest add yannickpschroeder/skills
```

### Manual

Copy a skill folder into `~/.claude/skills/` (user-wide) or `.claude/skills/` (per project).

### Then

Run `/setup-yannick-skills` once (per repo, or once for all projects) before your first `/autonomous` run.

## Design principles

- **Done means checkable.** Every phase ends with a "Done when" criterion a reviewer can verify.
- **Red before green.** A failure is pinned as a test before it is fixed; a test that never failed proves nothing.
- **Don't trust the measuring stick blindly.** Benches and oracles get audited, not just the system.
- **Ask rarely, park generously.** Blocking questions are reserved for decisions only the user can make.
- **Small and composable.** Each skill is a short Markdown file (plus a few templates where needed) you can read in five minutes and adapt.

## License

[MIT](./LICENSE) © Yannick Schröder
