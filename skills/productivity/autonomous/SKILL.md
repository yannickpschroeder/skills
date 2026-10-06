---
name: autonomous
description: Long autonomous run while the user is away — build a plan from the open items, then work through it block by block without asking; only true blockers stop the run, questions go to the parking lot. Usage: /autonomous [goal or plan file] [duration]
argument-hint: "[goal/plan file] [duration, e.g. 6h]"
disable-model-invocation: true
---

The user is away. From now on: **work, don't ask.** Every question that is not about a **true blocker** costs
the entire absence. Questions, doubts and decisions go to the **parking lot** and are presented only at the end.

## Terms

- **True blocker** — the ONLY thing that stops the run. All three must hold:
  1. The next step is irreversible, visible to the outside world or crosses a guardrail (pushing, deleting
     expensive data, switching production/workers live, changing golden test data or expected values without a
     decision, making a decision the user explicitly reserved for themselves), OR something is missing that only
     the user can provide (access, money, a domain judgement with no defensible assumption).
  2. There is no defensible **provisional assumption** under which work can continue.
  3. No other block of the plan can run instead.
  If any of the three does not hold, it is NOT a blocker → park it and move on.
- **Parking lot** — `<work folder>/open_questions.md`: per entry the question, context in plain language, the
  options with a recommendation, the chosen provisional assumption, and what depends on it. Never only in the chat.
- **Decision sink** — where real decisions are filed so the user can answer them outside the session (issues,
  a ticket system, decision files). Configured in the `## Agent skills` block of the project's CLAUDE.md/AGENTS.md,
  else of the user-level CLAUDE.md; the project block wins. Every parked entry that needs the user's decision is
  filed there **when it is parked**, not at the end, with the link noted in the parking lot. Filing into the
  configured sink is pre-authorized — it is not a guardrail crossing; anything the sink config doesn't cover
  (another repo, another tracker) still is. No sink configured → the parking lot is the only place, and the
  final message suggests running `/setup-yannick-skills`. Sink unreachable (auth, network, repo outside the
  configured target) → never a blocker: keep the entry in the parking lot marked `not filed: <reason>`, retry at
  wrap-up.
- **Work folder** — where plan, parking lot and progress log live: the folder the user names, else the project's
  convention (CLAUDE.md/memory), else `.autonomous/<start date of the run>/` in the repo root. Fixed once at the
  start and written as the first line of every progress update; after a compaction take it from there, never
  recompute it (a run past midnight would otherwise lose its plan).
- **Guardrails** — everything the user has ever forbidden or reserved for themselves: CLAUDE.md, memory
  (feedback notes), project docs. They apply unchanged during the run; "autonomous" grants no extra authority.
- **Progress log** — `<work folder>/report.md`, updated after EVERY block (status, numbers, commits, what comes
  next). It must survive a context compaction: after a compaction, the run first reads the plan and the log and
  continues exactly where they say.

## 1. Situation and plan

1. Determine the goal: the user's argument, otherwise the open items from plan documents, the decision sink,
   memory and the last report. Check the decision sink for decisions answered since the last run — they unblock
   parked work and overrule provisional assumptions. Check the current state before planning (commits, code, test
   data) — plan nothing that is already done.
2. Write a plan file (the harness's plan folder or `<work folder>/plan.md`), structured in **blocks**:
   - per block: goal, steps, **done criterion** (checkable: bench green, tests + mutation probes, commit,
     ledger line), estimated duration, dependencies;
   - order by dependency, independent blocks marked as parallel tracks;
   - a **buffer** at the end: work that only runs if time is left;
   - a guardrails section (spelled out concretely for this project) and an "already parked" section.
   The plan must last longer than the absence: better too many blocks than a run that idles after two hours.
3. If the user is still there, present the plan once (the skill's only question). If they are already gone,
   start immediately.

**Done when:** every block has a checkable done criterion, every dependency is named and the buffer covers the
planned duration.

## 2. Execute — block by block

For each block:
1. **Start:** long runs (suites, benches, rebuilds, external reviews) in the background; independent blocks in
   parallel to subagents (separate files per agent, never two agents on the same file). While a suite is running,
   edit nothing it reads.
2. **Build to project standard:** tests red first, then green; mutation probes for every new condition; benches
   before/after; every change explained. Never change golden data or expectations to get green — red is a finding.
3. **Review before committing:** an independent review as configured under Review in the `## Agent skills`
   block (otherwise a fresh reviewer subagent without the implementation's context), work in the findings with a
   pinning test, cross-check. The review replaces the
   question to the user — it is MANDATORY, not optional, precisely because nobody is watching.
4. **Finish:** commit following project convention (stage only your own files), measurement ledger, progress log.
5. **Next:** next block. A block that hangs on a non-blocker is finished under a provisional assumption or set
   aside until the decision (parking lot) — and the run moves on to the next block.

On failure: diagnose the cause, don't blindly repeat the same thing. After two failed attempts at the same
problem: park it, next block.

**Done when:** every block has either reached its done criterion, been parked with a reason or is stuck on a
true blocker — and the progress log says which for every block.

## 3. Wrap-up (always, even on abort)

1. Finish `report.md`: what ran (with commits and numbers), what stayed red, what is parked.
2. Review the parking lot: every question in plain language with context and examples, no abbreviations;
   every decision filed in the decision sink and linked — or, if filing still fails, listed in the final
   message as `not filed` with the reason.
3. Update memory (new state, decisions needed).
4. Final message: status in a few sentences, commits, open decisions as direct links, the next sensible step.

**Done when:** on return, the user knows from the final message and the report alone what happened and what they
need to decide — without reading the transcript.
