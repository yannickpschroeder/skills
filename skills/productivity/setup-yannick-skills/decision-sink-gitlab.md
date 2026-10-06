# Decision sink: GitLab

Decisions that only the user can make are filed as GitLab issues labelled `needs-decision`. Use the
[`glab`](https://gitlab.com/gitlab-org/cli) CLI.

## Target

`<target>` in every command below is this repo — never omit `--repo`.


- **Repo scope:** always `--repo <namespace>/<name>` (filled in by the setup) — never inferred, so a second remote or
  a fork can't redirect it.
- **User scope:** the current repo, but only if its namespace (user or group) is one of: `<owners filled in by the setup>`. Pass it
  explicitly with `--repo`. Any other repo (a cloned third-party project, no remote) counts as **unreachable** —
  never file decisions into someone else's tracker.

Filing into the target — including creating the `needs-decision` label there if it's missing — is authorized by
this setup: it is not a guardrail crossing, even during an unattended run.

## File a decision

`glab issue create --repo <target> --yes --label needs-decision --title "<the question, in plain language>" --description "..."`.
The description carries the fields the calling skill prescribes (context, options with a recommendation, the
provisional assumption taken, what depends on it). If the label is missing, create it first:
`glab label create --repo <target> --name needs-decision --description "Waiting for a decision by the maintainer"`.

## Check for answers

`glab issue list --repo <target> --label needs-decision --all`, then `glab issue view --repo <target> <id> --comments` for candidates. An issue
is **decided** when it is closed or the user has commented with a choice.

## Link

The issue URL printed by `glab issue create`.
