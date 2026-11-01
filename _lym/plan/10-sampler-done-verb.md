# Card 10 — sampler-done-verb

**Create:**
- `internal/loomcli#samplerDoneDeps`
- `internal/loomcli#samplerDoneVerb`
- `internal/loomcli#loomCLI.samplerDoneCmd`
- `internal/loomcli#TestSamplerDoneVerb`

**Edit:**
- `internal/loomcli#Command`

**Extends:**
- `internal/loomcli/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/loomcli#loomCLI.prContinueCmd`
- `internal/loomcli#prContinueVerb`
- `internal/loomcli#TestPRContinueVerb`
- `internal/loomcli#samplerStamp`
- `internal/loomrecipe#EngineRow`
- `internal/samplerengine#UpdateRecord`
- `internal/samplerengine#ReadRecord`

**Intent:**
D6: `lym loom sampler-done` writes the done flag that ends the Sampler-Session row, beside the entry stamp it computes.
`samplerDoneDeps` holds `row func() (string, bool, error)` (the recipe's SamplerSession row), `currentRow func() (string, error)` (the status file's current producer, empty with no status file), `recordPath` and `recordLockPath`, `stamp func() (int, error)` and `dirty func() ([]string, error)`.
`samplerDoneVerb` refuses, each through `output.Err` with a way forward (`PATTERN-refusal-way-forward`), in this order:
a recipe with no SamplerSession row, judged by its rows (`PATTERN-loom-contract`): "this run's recipe has no Sampler-Session row, so there is no session to end; way forward: none is needed, since the run's build ends on its own rows; `lym loom status` shows where it stands";
a run whose current row is not that row: naming the current row and "way forward: run lym loom sampler-done while the run stands at Sampler-Session; lym loom status shows the current row";
a record absent or with no recorded strand: "way forward: wait until the Sampler-Session row has started its session, then run lym loom sampler-done again; when the row halted because its session could not start, fix what its halt reason names and run lym loom resume, or end the run with lym batten cancel <slug>";
a task worktree with uncommitted paths, listing them: "way forward: commit the work, or discard what the operator does not keep, then run lym loom sampler-done again".
A failed read of any seam is an error envelope carrying the error and naming a re-run.
Otherwise it sets the record's `DoneStamp` to `stamp()` through `samplerengine.UpdateRecord` and reports `{"done": true, "stamp": <n>, "note": "the run ends Sampler-Session on a clean worktree and goes on to the Code-Review perch or Describe"}`; a second call writes the same stamp again.
Bound: the verb only writes a flag; the row ends only on the flag at its current stamp with a clean checkout and an intact record, and it is the only way the build segment ends besides a halt or `lym batten cancel`.
It is not limited to the agent's strand, so the operator's own shell may run it, and it is accepted while the run is halted at the row, so the next `lym loom resume` ends the row Done at once; a dirty checkout is refused here and never ends the row.
`samplerDoneCmd` mirrors `prContinueCmd`: `Use` `sampler-done`, the role audience annotation, no args, a `Long` text with an example, and deps built from `c.location` and `c.recipe`: `loomrecipe.EngineRow(c.recipe, "SamplerSession")`, the status file's current producer, `shedrun.SamplerRecord` and its lock, `samplerStamp` and `verifytree.DirtyPaths`.
`Command` adds it beside `prContinueCmd`.
`TestSamplerDoneVerb` (`//go:build tier0`, in `commitrecords_test.go` beside `TestPRContinueVerb`) is a table over fake deps and a temp record: a darn run (no row) is refused with its way forward; a sampler run at `Describe` is refused naming the current row; no record and a record with no strand are each refused; a dirty worktree is refused naming the path, with the record unchanged; a clean worktree with a recorded strand writes `DoneStamp` 3 when the stamp is 3 (the verb reads no run state, so a run halted at the row takes this same path); a repeat call succeeds with the same stamp; a read failure of each seam, `row`, `currentRow`, the record, `stamp` and `dirty`, is an error envelope carrying the error, with the record unchanged (D6 acceptance on the refusals).
`loomcli/doc.go` gains the verb's sentence beside `pr-continue`'s, and `docs/overview.md` names it with the sampler recipe.

**ImpactSummary:** one verb added to loom's command list; no existing verb changes.

**Commit:** 10: lym loom sampler-done
