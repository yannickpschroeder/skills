# Decision sink: local markdown

Decisions that only the user can make are filed as markdown files under `docs/decisions/`, one file per
decision, committed with the work that raised them.

## File a decision

Create `docs/decisions/NN-<slug>.md` (numbered from `01`, next free number). First lines:

```
# <the question, in plain language>
Status: open
```

Below that, the fields the calling skill prescribes (context, options with a recommendation, the provisional
assumption taken, what depends on it).

## Check for answers

Scan `docs/decisions/` for files with `Status: decided`. The user records the answer under an `## Answer`
heading and flips the status.

## Link

The file path relative to the repo root.
