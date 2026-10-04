# Board as one list: kind, labels and GitHub intake

The board becomes the operator's one list: tasks and notes, both labelled, with GitHub issues pulled in as notes.
GitHub stays the inbox, because `lyx selfreport` files from any repo lyx runs in; the board is loomyard's plan.

## What changes

- `kind` (`task` or `note`) replaces `tier` on every entry.
  A new entry without `kind` is a note, and only a task is claimable.
  Tier 2 and "Next Up" are gone from the model, the CLI, the README and the `ly:board` skill.
- `labels` replaces `type` and the bracketed brief prefixes.
  `_lyx/config/board.yaml` gains two lists, `types` (default `[bug, enhancement]`) and `labels` (default `[undecided]`); a label is valid when it is in either.
  A note carries exactly one type label, a task one or more.
  Duplicates and unlisted labels are refused, naming the entry, the label and `board.yaml`.
- `promote` changes a note into a task; on a task it returns the entry unchanged, and a `tier` key is refused.
- `depends_on` belongs to tasks only, and every target must be a task.
  Demoting a task that has or is a dependency is refused, with the way forward.
- `lyx board list` and `find` take a repeatable `--label`, combined with AND; `--text` lines show kind and labels.
  A flag label is refused only when no config list and no entry carries it, so entries holding a removed label stay listable for relabelling.
- The README renders Tasks by dependency layer, Notes under one section per configured type (`Other` for an orphaned type), then Done, with labels on every line.
- `lyx board upsert --body-file <path>` (`-` for stdin) supplies the body; a payload that also has `body`, or that itself arrives on stdin, is refused.
- Every entry carries `issues`, a list of inbox issue numbers settable through `upsert`.
- `lyx board intake` has three verbs against the `selfreport` inbox repo:
  - `list` shows open issues, pull requests excluded, that no entry records.
  - `import` creates a note from an issue, or with `into` folds it into an existing entry; it then comments and closes the issue as completed.
  - `close` closes an issue with a required reason, as not planned or, with `completed`, as completed.
- The hidden `notes` group and `promote-note` verb are removed, with their tests and `cliTreeFindings` entries.

## Design decisions

- `import` never overwrites: a `slug` that names an entry, or an `into` that names none, is refused before any board write or GitHub call.
  A recorded issue makes `import` a no-op that names the recording entries.
- The recorded-issue checks and the board write share one lock acquisition; the GitHub comment and close run after the lock is released.
  If the GitHub step fails, the report carries the exact `close` payload to run.
- A fold adds only the issue's GitHub labels the config lists and the target lacks, never a type label to a note; the rest are reported as dropped.
- `selfreportengine` stays the sole owner of the inbox repo constant and gains the list, fetch, and comment-and-close calls through `NewGitHubClient`; auth stays `githubclient.New`.
- Old-shape `board.json` entries convert in memory on load and persist on the next write: tier 1 becomes a task, other tiers notes; `bug` stays `bug`, `feature` and `chore` become `enhancement`, `design` becomes `enhancement` plus `undecided`.
  Bracketed brief prefixes become labels and are stripped.
  A migrated note's `depends_on` moves into its body as a closing line.
  Loading never validates labels against config; the next write of an entry with an unlisted label is refused.
- The design moves from the board entry into `internal/boardengine/doc.go`; `docs/overview.md`, the `ly:board` skill and the sandbox suite doc are updated.

## Before deploying

The hub must write this list into the parent's live `_lyx/config/board.yaml` before deploying the landed binary.
Until then the binary refuses the next write to every migrated entry that carries an area label.

```yaml
labels: [undecided, agents, batcher, batten, board, bouncer, burler, cli, config, cost, discussion, docs, engines, fabric, finalize, github, glyph, ide, landing, loom, messaging, models, orch, parent-review, perf, permissions, plan, plugins, process, quarry, recipes, reed, review, search, shed, shuttle, slack, speed, testing, ui, vocabulary, webster, windows]
```

After writing it, delete the matching paragraph in `internal/boardengine/doc.go`.
A `lyx` process started before the deploy that writes `board.json` after migration drops `kind`, `labels` and `issues`, so deploy between runs.

## To check by hand

- `lyx board intake list`, `import` and `close` against the real inbox repo; the suite covers them only against an `httptest` server.
- A migration dry run on a copy of the live `board.json`, confirming the label list above covers every migrated entry.
