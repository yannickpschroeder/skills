# Decision sink: GitHub

Decisions that only the user can make are filed as GitHub issues labelled `needs-decision`. Use the `gh` CLI.

## Target

`<target>` in every command below is this repo — never omit `--repo`.


- **Repo scope:** always `--repo <owner>/<name>` (filled in by the setup) — never inferred, so a second remote or
  a fork can't redirect it.
- **User scope:** the current repo, but only if its owner is one of: `<owners filled in by the setup>`. Pass it
  explicitly with `--repo`. Any other repo (a cloned third-party project, no remote) counts as **unreachable** —
  never file decisions into someone else's tracker.

Filing into the target — including creating the `needs-decision` label there if it's missing — is authorized by
this setup: it is not a guardrail crossing, even during an unattended run.

## File a decision

`gh issue create --repo <target> --label needs-decision --title "<the question, in plain language>" --body "..."` — use a
heredoc for the body. The body carries the fields the calling skill prescribes (context, options with a
recommendation, the provisional assumption taken, what depends on it). If the label is missing, create it first:
`gh label create --repo <target> needs-decision --description "Waiting for a decision by the maintainer"`.

## Check for answers

`gh issue list --repo <target> --label needs-decision --state all --json number,title,state,comments,url`. An issue is
**decided** when it is closed or the user has commented with a choice; read the comments for the answer.

## Link

The issue URL from `gh issue create` / `--json url`.
