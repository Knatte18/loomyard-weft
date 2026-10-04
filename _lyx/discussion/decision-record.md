## Goal

The board becomes the operator's one list: two kinds of entry, tasks and notes, both labelled, with GitHub issues pulled in as notes so nobody tracks GitHub and the board side by side.
GitHub stays the inbox, because `lyx selfreport` files from any repo lyx runs in and only GitHub is reachable from all of them;
the board is loomyard's plan.

## Scope

In:

- A `kind` field (`task` or `note`) replaces `tier` on every entry; tier 2 and "Next Up" leave the model, the CLI, the README and the `ly:board` skill.
- A `labels` field on every entry replaces `type` and the bracketed brief prefixes; type labels and other labels are validated against two lists in the board config.
- `promote` changes `kind` from `note` to `task`.
- `lyx board list` and `lyx board find` filter by label; the JSON and `--text` listings show kind and labels.
- The README renders Tasks (by dependency layer, as today) and Notes grouped by type label, with labels on every entry.
- `lyx board upsert` reads the body from a file or stdin.
- `lyx board intake`: list open inbox issues not yet on the board, import one as a note or into an existing entry and close it with a pointer, close noise with a stated reason.
- On-load migration of existing `board.json` entries to the new shape.
- Removal of the hidden `notes` alias group and the hidden `promote-note` alias, whose vocabulary the new `kind` contradicts.
- Docs in the same commit: `boardengine`'s package doc, the board line in `docs/overview.md`, the `ly:board` skill, the `upsert`/`promote`/`list`/`find` help text.

Out:

- Automatic triage: the intake verb never decides what is noise, a duplicate or a fold target; the orchestrator does.
- Any inbox other than the one repo `selfreport` files to.
- Kind or label filters on verbs other than `list` and `find`.
- Curating the migrated area labels into a smaller set; the operator does that later with one config edit and relabelling.

## Decisions

### D1 — `kind` replaces `tier`

Decision: every entry carries `kind`, `task` or `note`; the `tier` field, `MinTier`/`MaxTier`/`DefaultTier` and the tier-ordering rule are removed.
A new entry without `kind` is a `note`.
Only a task is claimable, as only tier 1 was.
Rationale: tier 2 was unused and the board has run as tasks plus notes since 2026-10-03; the operator decided the two-layer model on 2026-10-04.

### D2 — `depends_on` belongs to tasks only

Decision: a note's `depends_on` must be empty, and every `depends_on` target must be a task (done or open).
Changing a task to a note is refused while it has its own `depends_on` or while another entry depends on it.
Each refusal names the way forward: clear the list with `lyx board set-deps`, or keep the entry a task.
A relation between notes is a sentence in a body.
Rationale: this replaces the tier rule (a dependent's tier ≥ its dependency's) with its two-kind equivalent, so the README's dependency layers stay meaningful for tasks alone.

### D3 — one `labels` field carries type and area

Decision: `labels` is a list of strings on every entry, replacing `type`.
The board config (`_lyx/config/board.yaml`) gains two lists: `types` (template default `[bug, enhancement]`) and `labels` (template default `[undecided]`).
A label is valid when it appears in either list; anything else is refused with a message naming the label, the entry and the way forward (add it to `board.yaml`'s list, or drop it from the entry).
A note carries exactly one type label;
a task carries one or more, because it carries the type labels of the notes it bundles.
`undecided` marks an entry whose what-to-build is not yet decided and is removed once it is.
`labels` replaces the whole list on upsert; order is preserved as given and duplicates are refused.
Validation runs on the full resulting entry of every write that creates or changes an entry, so an entry left holding a label since removed from config is refused on its next write until relabelled; reads never validate.
Rationale: one classification on two axes, changeable by the operator with one config edit, as decided on 2026-10-04.
Bound: validation applies only to writes through a `Board` built from config; the `--board-path` shape already refuses every write, and read-only `Board`s built from a path alone (batten's recipe read) never validate.

### D4 — `promote` sets `kind: task`

Decision: `lyx board promote '{"slug":…}'` (or `id`) changes a note into a task and validates the result like any write; the `tier` key is gone from its payload.
Promoting an entry that is already a task returns it unchanged.
Demotion goes through `upsert` with `kind: note`, under D2's refusals.
Rationale: with two kinds, promotion has one target.

### D5 — the `notes` and `promote-note` aliases are removed

Decision: the hidden `lyx board notes` group and the hidden `lyx board promote-note` verb are deleted, with their tests and their two `cliTreeFindings` entries in `cmd/lyx/clitree_test.go`.
Rationale: both are pre-merge back-compat aliases no prompt, skill or script calls, and "notes" now names a kind, so an alias that reaches every entry under that name misleads.

### D6 — listing and filtering

Decision: `BriefTask` and `--text` output carry `kind` and `labels` in place of `tier` and `type`.
`list` and `find` take a repeatable `--label <name>` flag; an entry matches when it carries every named label.
An unknown label in the flag is refused, as on write.
The `--text` line shows kind, slug, title, the labels joined by commas, and status as today.
Rationale: label filtering is the operator's decided way to see one area; refusing an unknown filter label catches typos that would otherwise return an empty list silently.

### D7 — README layout

Decision: the README renders `## Tasks` split into dependency layers as today, then `## Notes` with one `###` section per type in the config's `types` order, its heading the type name capitalised and pluralised (`Bugs`, `Enhancements`), then `## Done`.
Every entry line shows its labels.
A note whose type label is no longer in config renders under a final `### Other` section, so a config edit never drops an entry from the README.
Rationale: the operator decided notes group by type; the README cannot filter, so labels on each line serve area scanning.

### D8 — upsert body from a file or stdin

Decision: `lyx board upsert` takes `--body-file <path>`, where `-` reads stdin; the file's content becomes the payload's `body`.
A payload that also carries `body` is refused, naming both sources.
Rationale: editing a body no longer means splicing it into a JSON argument in a shell one-liner.
Bound: the flag only supplies `body`; every other field still comes from the JSON payload and passes the same validation.

### D9 — `issues` field

Decision: every entry carries `issues`, a list of inbox issue numbers, settable through `upsert` so merging notes into a task can carry their numbers.
Rationale: a back-link from the board to GitHub, and the record `intake` uses to tell an imported issue from a new one.

### D10 — `lyx board intake`, a mechanical verb group

Decision: three verbs under `lyx board intake`, all against the inbox repo, all JSON in and out:

- `list` — every open issue (pull requests excluded, all pages) whose number no board entry records, with number, title, body, labels, URL and creation time.
- `import '{"issue":N,"slug":…,"title"?,"brief"?,"labels"?}'` creates a note; `import '{"issue":N,"into":"<slug>"}'` folds the issue into an existing entry instead.
  A new note's body is the issue body under a link back to the issue; a fold appends a `## From issue #N` section with the same content to the target's body.
  Either way `N` is added to the entry's `issues`.
  Without `labels`, the note takes the issue's GitHub labels that the config lists, and the result reports the dropped ones; the result must still pass D3, so an issue with no usable type label is refused with the way forward (pass `labels`).
  After the board write, the issue gets a comment naming the entry's slug and is closed as completed.
- `close '{"issue":N,"reason":"…"}'` comments the reason and closes the issue as not planned; `reason` is required and non-empty.

Sequencing: the issue is fetched first, the board write runs under the board lock, and the GitHub close runs after the lock is released, so no network call is made while holding the lock.
Rationale: the verb does the mechanics, the orchestrator triages, as decided; the first intake (2026-10-04, by hand) showed these three moves are the whole procedure.
Bound: `import` of an issue some entry already records is refused when the issue is closed, and when it is still open (a previous import whose close failed) it only retries the comment and close, writing nothing to the board.
`intake` never deletes or edits an issue's content and never touches a repo other than the inbox.

### D11 — the inbox repo and auth are `selfreport`'s

Decision: `selfreportengine` stays the sole owner of the inbox repo constant and gains the list, fetch, comment-and-close calls `intake` needs, through the same `NewGitHubClient` seam; `boardcli` wires `intake` from `boardengine` (the store-side import and fold) and `selfreportengine` (the GitHub side).
Authentication is `githubclient.New`, unchanged.
Rationale: the inbox is by definition where `selfreport` files, so one constant serves both directions; PATTERN-github-auth is met by construction.

### D12 — on-load migration

Decision: an entry decoded from `board.json` with `tier` and no `kind` converts in memory on load and persists on the next write, as the legacy `tasks.json` migration does:

- tier 1 → `task`; tier 2 and 3 → `note`.
- `type`: `bug` → `bug`; `feature` and `chore` → `enhancement`; `design` → `enhancement` plus `undecided`.
- Leading `[x]` tokens in the brief become labels in order, and are stripped from the brief; a token equal to a type name is dropped, since the type comes from `type`.
- A migrated note's `depends_on` moves into its body as a closing line `Depends on `a`, `b`.` and the list is emptied.
- `issues` starts empty.

Migration does not validate labels against config, so it never fails; D3's write-time refusal then applies.
The legacy `tasks.json`/`notes.json` path chains into this conversion.
Rationale: every hub lyx runs in migrates without an operator step, and nothing is lost.

### D13 — loomyard's own area list

Decision: the plan derives every area label the migration produces on loomyard's live board and puts the exact `labels` list, as a ready-to-paste `board.yaml` snippet, into the PR description.
The hub writes that list into the parent's live `_lyx/config/board.yaml` before deploying the landed binary, so no migrated loomyard entry is refused on its next write.
The pair may add the same list to its own `_lyx/config/board.yaml` for its own runs, knowing that file is weft content that never reaches the parent.
Rationale: the template stays generic for other repos, and loomyard's own vocabulary is loomyard's config;
`_lyx/` in a pair links into the pair's weft worktree, which does not merge at landing, so the PR description is the channel that reaches the hub.
Bound: no weft-carry mechanism is added at landing.

## Constraints

- PATTERN-config-strictness: `boardengine` stays a strict `configengine.Load` caller; the new lists are template defaults, not minimums.
- PATTERN-github-auth: GitHub is reached only through `githubclient`; no `gh` shell-out.
- PATTERN-cli-cobra: every new command has a `Short`, errors are JSON via `internal/output`, and every `RunE` checks `clihelp.ShouldAbort`; the clitree walk covers `intake` with no new findings entry.
- PATTERN-test-tier-purity: GitHub calls are tested against an `httptest` server through `NewGitHubClient`, as `selfreportengine`'s suite does, untagged.
- PATTERN-lyxdirs-single-declarer and PATTERN-cwd-resolution: `intake` resolves the board the way every board verb does today, through the existing `PersistentPreRunE`.
- PATTERN-documentation-lifecycle: the intake and label design moves from this board entry into `boardengine`'s package doc; `docs/overview.md`'s board line is updated.
- `board.json` stays the authoritative store; the README is derived.
- `_board/` is touched only through `lyx board`.

## Auto-mode assumptions

- The inbox repo is `selfreport`'s target; no per-repo inbox config is added (D11).
- `undecided` is a plain label in the config's `labels` list, not a third axis.
- Type headings derive from the type name (capitalised, plural) rather than a configured mapping.
- `--label` filters combine with AND.
- Removing the `notes` and `promote-note` aliases is in scope, since the `kind` vocabulary would otherwise contradict them.
- Migrated area labels are carried as-is, without merging near-duplicates.
- `isolated`, `recipe`, `short_name` and `status` keep their current meaning on both kinds.

## Open risks

- A `lyx` process started before deploy that writes `board.json` after a new binary migrated it decodes neither `kind`, `labels` nor `issues` and drops them on save.
  The operator deploys through `update-plugins.sh` between runs; this task's own landing runs on the old binary against the unmigrated store, so it is unaffected.
- Until the hub writes D13's `labels` list into the parent's live `_lyx/config/board.yaml`, the deployed binary refuses the next write to every migrated entry carrying an area label; the hub must write the list from the PR description before deploying.
- Config lives per worktree, so a task branch cut before the operator adds a label refuses that label until it merges the newer config; the refusal names the config file.
- A GitHub close that keeps failing leaves an issue open and recorded; `intake list` does not show it, and only a retried `import` closes it.

## Acceptance criteria

- `board.json` entries in the old shape load as D12 says, for every tier and type, with brief prefixes stripped into labels and a note's dependencies moved into its body; the next write persists the new shape and drops `tier` and `type`.
- An upsert with a label outside both config lists, a note with zero or two type labels, a task with no type label, or duplicate labels is refused with a message naming the label or entry and the config file.
- A note with `depends_on`, a dependency on a note, and demoting a task that has or is a dependency are each refused with the way forward.
- `promote` turns a note into a task; on a task it returns the entry unchanged; a payload with `tier` is refused as an unknown key.
- `list --label a --label b` and `find --label a <text>` return only entries carrying every named label; an unknown label is refused; `--text` lines show kind and labels.
- The README renders Tasks by layer, Notes under one section per configured type in config order plus `Other` for an orphaned type, and Done; every entry line shows its labels; no "Next Up" or tier wording remains.
- `upsert --body-file <path>` and `--body-file -` set the body; a payload with `body` plus the flag is refused.
- `intake list` excludes pull requests, issues recorded in any entry's `issues`, and closed issues, and follows pagination.
- `intake import` creates a note with the link-back body, carried-over config labels and the issue number, then comments and closes as completed; with `into` it appends the section to the target and records the number; with no usable type label it is refused before any board write or GitHub mutation.
- `intake import` of a recorded, still-open issue writes nothing to the board and only closes it; of a recorded, closed issue it is refused.
- A GitHub failure after the board write reports the written entry and the retry command.
- `intake close` without a reason is refused; with one it comments and closes as not planned.
- `lyx board notes` and `lyx board promote-note` no longer exist, and `cliTreeFindings` has no `board notes` entries.
- The PR description carries the exact `labels` list for the parent's `board.yaml`, covering every area label the migration yields on loomyard's live board.
- The clitree walk passes for every new command; `go test ./...` passes.

## Notes for the plan writer

- `internal/boardengine/task.go`, `store.go` (`validateWrite`, `validateTier`, `Promote`, `ListTasksBrief`, `Find`, `upsertAllowedKeys`), `render.go` (`tierSections`, `renderTasksSection`, `metaLineWithSlug`), `legacy.go`, `config.go` and `template.yaml` carry the model.
- `internal/boardcli/cli.go` and `text.go` carry the verbs and the compact listing; `promotenote_test.go` and the `notes` alias tests go with D5.
- `internal/selfreportengine/selfreport.go` holds `targetRepo` and the `NewGitHubClient` seam D11 extends.
- `plugins/ly/skills/board/SKILL.md` describes tiers and types and the "never name a dependency in a body" rule, which now holds for tasks only.
- `internal/battencli/wire.go` reads `Recipe` from a path-only `Board`; it must keep working without config.
- `internal/ideengine/menu.go` and `internal/loomcli/landingdeps.go` open config-built `Board`s.
