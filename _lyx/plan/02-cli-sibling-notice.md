# Card 2 — cli-sibling-notice

**Create:**
- `internal/boardcli#siblingNotice`
- `internal/boardcli#joinNotices`
- `internal/boardcli#outputSuccessWithNotice`
- `internal/boardcli_test#stepSiblingWarning`

**Edit:**
- `internal/boardcli#storeVerbs`
- `internal/boardcli_test#TestCLI`
- `internal/boardcli_test#TestHelpSchema_LeafCommands`
- `internal/boardcli/doc.go`
- `plugins/ly/skills/board/SKILL.md`

**Uses:**
- `internal/boardcli#outputSuccessWithTask`
- `internal/boardcli#outputSuccessWithCount`
- `internal/boardcli_test#runJSON`
- `internal/boardcli_test#mustUpsert`
- `internal/boardcli_test#seedHubBoardConfig`

**Intent:**
The warning of decisions D3 and D4, and the skill rule of D5.

`siblingNotice`, in `internal/boardcli/cli.go`, formats card 1's siblings as one line per pair, joined by newlines, empty for none:
`"<X>" now waits on <D, ...>, which "<Y>" also waits on, and the two share module labels <L, ...>: chain one after the other with "lyx board set-deps", or leave both if they do not overlap`.
`joinNotices` joins its non-empty arguments with newlines.
A run-settings notice and a sibling line together are reachable only through `upsert-batch`: the run-settings notice needs the upserted entry held past Seed-Child, and the run lock refuses a `depends_on` change on a held entry, so on `upsert` and `merge`, whose one upserted entry is both, `joinNotices` never receives two non-empty parts.
`outputSuccessWithNotice` writes `{"ok":true}` plus a `notice` field when the notice is not empty, so a `set-deps` without a sibling still prints exactly `{"ok":true}`.

In `storeVerbs`:
- `upsert` calls `Board.UpsertTaskSiblings` and prints `joinNotices(run-settings notice, siblingNotice(siblings))` as the task envelope's `notice`.
- `upsert-batch` appends `siblingNotice` of the batch's siblings after the per-task run-settings notices.
- `merge` joins its run-settings notice with the merge's `siblingNotice`.
- `set-deps` prints through `outputSuccessWithNotice`.
- A refused write prints the same error envelope as today and no notice; the warning changes no exit code, no refusal and no stored record.
- The `Long` of the four verbs gains one sentence: the write still succeeds when it gives a task a new `depends_on` entry another open task waits on and the two share a module label, and the envelope's `notice` names the sibling, the dependency and the labels.

`stepSiblingWarning`, in `internal/boardcli/cli_test.go` beside the other store-verb steps and registered in `TestCLI` after `RunSettings`, seeds a `board.yaml` whose `labels` hold `board` through `seedHubBoardConfig`, seeds every entry as a `"kind":"task"` carrying `bug`, and asserts:
- each of `set-deps`, `upsert`, `upsert-batch` and `merge` adding D to X, with Y open on D sharing `board`: exit 0, and `notice` contains X, Y, D, `board` and `lyx board set-deps`; for `set-deps` the stored `depends_on` is read back with `get`.
- `set-deps` adding D to X while Y1 and Y2 both wait on D sharing `board`: `notice` is two lines, one naming Y1 and one naming Y2.
- `set-deps` with no sibling: the envelope has no `notice` key.
- `upsert-batch` whose one element changes `run_settings` on an entry held past Seed-Child (`running · Discussion-Write`) and whose other element adds D to X with sibling Y: `notice` holds both the `--layer local` run-settings line and the sibling line.

A refused write goes through `outputError`, which carries no `notice`, and card 1's refused rows pin that a refusal returns no siblings, so this step plans no refused case.

`TestHelpSchema_LeafCommands` adds `notice` to the `mustContain` of the `upsert`, `upsert-batch`, `merge` and `set-deps` rows, and its `//testtiming:keep` reason gains the sibling-warning sentence among what it pins.

`internal/boardcli/doc.go` gains a paragraph: `set-deps`, `upsert`, `upsert-batch` and `merge` add the sibling warning to the success envelope's `notice`, joined after a run-settings notice, one line per pair, a batch pair once; `set-deps` prints `notice` only then; `promote` and `intake import` write no `depends_on` and stay unchanged.

`plugins/ly/skills/board/SKILL.md`'s `## Writing a body` paragraph on `depends_on` gains the rule: whenever you set `depends_on`, check the other tasks waiting on the same entry for overlap, not only the running ones, and act on the `notice` warning `set-deps` and the upsert verbs print by chaining overlapping tasks with `lyx board set-deps`.
The same edit puts "A relation between notes has no `depends_on`, so it is a sentence in a body." on its own line.
No test pins the skill's wording.

**ImpactSummary:** `storeVerbs` only, plus its help and doc text; output changes only by an added `notice` when a sibling exists, so no existing assertion changes.

**Commit:** 2: board: warn in the write envelope when a new dependency joins an overlapping sibling
