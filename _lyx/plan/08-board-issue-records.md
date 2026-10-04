# Card 8 — board-issue-records

**Edit:**
- `internal/boardengine/task.go`
- `internal/boardengine/store.go`
- `internal/boardengine/board.go`
- `internal/boardengine/store_test.go`

**Create:**
- `internal/boardengine/intake.go`
- `internal/boardengine/intake_test.go`

**Uses:**
- `internal/boardengine/labels.go`
- `internal/boardengine/config.go`

**Intent:**
The board side of intake: the `issues` field (D9) and the store-side import and fold of D10.
`boardengine` imports nothing GitHub-specific; `boardcli` (card 9) converts a fetched issue into this package's input type.

`issues` (D9):
- `Task` gains `Issues []int` (`json:"issues"`), empty by default, so an entry loaded without it, migrated or not, has an empty list.
- `upsertAllowedKeys` gains `issues`; it replaces the whole list as `labels` does, and the write refuses a duplicate number and a number that is not a positive integer.
  Bound: a number is not checked against GitHub.

`intake.go`:
- `InboxIssue`: number, title, body, URL, label names, whether it is open, whether it is a pull request.
- `ImportRequest`: the issue, plus either `Slug` with optional `Title`, `Brief` and `Labels` (a nil `Labels` means "not given"), or `Into`.
- `ImportResult`: the written entry, the slugs of the recording entries when the import was a no-op, and the issue labels dropped.
- `Store` gains `RecordingEntries(number int) []string` and `RecordedIssues() map[int][]string`.
- `Board.RecordedIssues()` is a read that persists nothing.
- `Board.ImportIssue(req ImportRequest) (ImportResult, error)` runs in one `boardCriticalSection`, so the recorded-issue, slug and `into` checks and the board write share one lock acquisition, and no network call is made under it:
  1. a pull request is refused;
  2. an issue that any entry's `issues` lists is a no-op returning the recording slugs, through `noWrite`, whatever else the request carries;
  3. a closed issue is refused;
  4. `Into` together with `Title`, `Brief` or `Labels` is refused, as is a request with both or neither of `Slug` and `Into`;
  5. a `Slug` an entry already names is refused, naming that entry (pick another slug, or fold with `into`);
     an `Into` naming no entry is refused (pick an existing slug, or import as a new note with `slug`);
  6. a new note's body is the line `Imported from [issue #N](<url>).`, a blank line, then the issue body; its title defaults to the issue title; its `issues` is `[N]`.
     Without `Labels`, it takes the issue labels the vocabulary knows and reports the rest dropped;
     more than one configured type label is refused naming them, and none is refused, each with the way forward (pass `labels`).
  7. a fold appends `## From issue #N`, a blank line and the same link-and-body content to the target's body, appends `N` to its `issues`, and adds the issue labels the vocabulary knows that the target does not already carry, never adding a type label to a note.
     The labels it does not add because they are unconfigured or a type label on a note are reported dropped, never refused.
     A target of either kind and any status is accepted.
  8. the resulting entry passes `validateWrite`, so a fold into a target that holds a label the config no longer lists is refused with D3's message.
  Every refusal happens before the save, so a refused import writes nothing.

Tests (`intake_test.go`, untagged, a `Board` over a temp dir with `SkipGit` and a configured `Vocabulary`; `store_test.go` for `issues`): `issues` replaced on upsert and duplicates and non-positive numbers refused;
a new note with the link-back body, carried-over labels, dropped labels reported and `issues: [N]`;
explicit `labels`; no usable type label and two type labels, each refused naming the labels;
a fold into a task and into a done entry, labels added only when configured and not carried, a type label never added to a note, the section appended;
a fold into a target with an unconfigured label refused;
a recorded issue (recorded by an earlier import and one recorded by hand through `upsert`) returned as a no-op naming the recording entries with `board.json` unchanged;
a taken slug, a missing `into`, `into` with `title`, a pull request and a closed unrecorded issue, each refused with `board.json` unchanged.

**ImpactSummary:** Adds one field to `Task` and one allowlisted upsert key; existing entries load with an empty `issues` and no existing verb changes behaviour.
