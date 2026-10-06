# Review

Before committing a block of work that nobody reads along, get it independently reviewed this way.

## Method

**Fresh subagent.** Spawn a reviewer subagent with no context from the implementation. Give it the diff, the
block's goal and done criterion, and the project's guardrails; ask for concrete findings (input/state → wrong
behaviour), not style opinions.

<!-- Replace or extend for a second model / CLI, e.g.:
**Second model.** Run `<exact command>` on the diff before the subagent; treat both outputs as findings. -->

## Handling findings

Pin every real finding with a failing test before fixing it, then cross-check the fix. A finding that turns out
wrong is noted with the reason, not silently dropped.
