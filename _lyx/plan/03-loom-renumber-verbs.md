# Card 3 — loom-renumber-verbs

**Create:**
- `plan:internal/websterengine#LastBegunCard` -> `func LastBegunCard(st *State) int`
- `plan:internal/loomcli#loomCLI.insertCardCmd` -> `func (c *loomCLI) insertCardCmd() *cobra.Command`
- `plan:internal/loomcli#loomCLI.removeCardCmd` -> `func (c *loomCLI) removeCardCmd() *cobra.Command`
- `plan:internal/loomcli#loomCLI.restoreRenumberCmd` -> `func (c *loomCLI) restoreRenumberCmd() *cobra.Command`
- `plan:internal/loomcli#loomCLI.underRenumberLease` -> `func (c *loomCLI) underRenumberLease(op func(r planparser.Renumber) (map[string]any, error)) (map[string]any, error)`
- `plan:internal/loomcli#renumberRefusal` -> `func renumberRefusal(verb string, err error) string`
- `internal/loomcli/renumber_test.go`

**Edit:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#loomCLI.resolveRunID`
- `internal/loomcli#genericShedVerb`
- `internal/loomcli#Arm`
- `internal/loomcli#TestVerbUsesLightweightWiring`
- `internal/loomcli#TestResolveRunID`
- `internal/loomcli/doc.go`
- `contracts/specs/refusal-spec.md`
- `contracts/specs/loom-plan-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/planparser#Renumber`
- `plan:internal/planparser#RenumberFS`
- `plan:internal/planparser#RenumberJournalPath`
- `plan:internal/planparser#Renumber.InsertCard`
- `plan:internal/planparser#Renumber.RemoveCard`
- `plan:internal/planparser#Renumber.Restore`
- `plan:internal/planparser#ErrRenumberJournalPresent`
- `plan:internal/planparser#ErrPlanUnparsed`
- `plan:internal/planparser#ErrPlanInconsistent`
- `plan:internal/planparser#ErrPositionOutOfRange`
- `plan:internal/planparser#ErrPositionFrozen`
- `plan:internal/planparser#ErrOnlyCard`
- `plan:internal/planparser#ErrSlugInvalid`
- `plan:internal/planparser#ErrIntentInvalid`
- `plan:internal/planparser#ErrRenumberHalted`
- `plan:internal/planparser#ErrNoRenumberJournal`
- `plan:internal/planparser#ErrRenumberJournalUndecodable`
- `plan:internal/planparser#ErrRenumberJournalOutside`
- `plan:internal/planparser#ErrRestoreIncomplete`
- `internal/planparser#PlanDir`
- `internal/websterengine#AcquireStateMutation`
- `internal/websterengine#LoadState`
- `internal/websterengine#Dir`
- `internal/websterengine#ScratchDir`
- `internal/websterengine#SaveState`
- `internal/websterengine#BatchState`
- `internal/websterengine#cardNumberInt`
- `internal/lock#TryAcquireWriteLock`
- `internal/clihelp#RenderIndex`
- `internal/loomcli#planFixture`
- `internal/testkit/plankit#Write`
- `internal/testkit/plankit#WriteTree`

**Intent:**
Mount three flat `lyx loom` verbs beside `validate-plan`, in a new `internal/loomcli/renumber.go` (decision record D1, D2, D3): `insert-card <N> <slug> --intent "<one-line intent>"`, `remove-card <N>` and `restore-renumber`.
Each carries `Use`, `Short`, a `Long` with an example, `Annotations` naming `clihelp.AudienceOperator` (so `lyx help index`, the index the `ly:cli` skill loads, lists all three with their `Short`), `cobra.ExactArgs(2)`, `ExactArgs(1)` or `NoArgs`, and a required `--intent` flag on `insert-card` (`PATTERN-cli-cobra`).
A non-integer `<N>` is a usage refusal from the verb's own `strconv.Atoi` check, worded `loom: <verb>: card number <arg> is not an integer; way forward: re-run with a card number`.
Each reports one JSON envelope through `output.Ok` or `output.Err`.

`websterengine.LastBegunCard`, beside `DoneCards` in `internal/websterengine/beginbatch.go`, returns the highest card number among the card ids of every batch record `st` holds, terminal or not, reading a record with no `Cards` as the single card its key and `Slug` name (the same fallback `DoneCards` applies) through `cardNumberInt`; a nil state or no record returns `0`.
This is the begun set the record's D2 names, not `DoneCards`'s terminal-done set.

`underRenumberLease` acquires `websterengine.AcquireStateMutation(websterengine.ScratchDir(c.env.AnchorPath))`, loads the run record with `websterengine.LoadState(websterengine.Dir(...), websterengine.ScratchDir(...))`, builds `planparser.Renumber{PlanDir: planparser.PlanDir(c.env.AnchorPath), JournalPath: planparser.RenumberJournalPath(c.env.AnchorPath), FrozenThrough: websterengine.LastBegunCard(st), FS: c.renumberFS}`, runs `op` and releases the lease after `op` returns, so the lease spans the run-record read through the last apply step and a begin-batch blocks on it meanwhile.
`restore-renumber` holds the same lease.
`loomCLI` gains one field, `renumberFS planparser.RenumberFS`, nil in production and set by tests, documented like the struct's other seams.
None of the three verbs commits, approves or touches `approved:`; the Plan-Burler row's commit seam commits the plan directory as today, deletions included.

Success envelopes: `insert-card` returns `plan_dir`, `created`, `moved` (each `{from, to, old_file, new_file}`) and `prose` (each `{file, line, text}`); `remove-card` returns the same with `removed` in place of `created`; `restore-renumber` returns `plan_dir`, `removed` and `restored`.
`moved` gives a standalone `lyx webster rebaseline` every number it must name with `--card`, and the verbs change no overview section but the Card Index, so the frame hash rebaseline compares is unchanged.

`renumberRefusal` maps each error to its message by `errors.Is`, each ending in the `way forward:` clause its `refusal-spec.md` row quotes:

| Sentinel | Way forward |
|---|---|
| `ErrRenumberJournalPresent`, `ErrRenumberHalted` | run `lyx loom restore-renumber` |
| `ErrPlanUnparsed`, `ErrPlanInconsistent` | fix what `lyx loom validate-plan` reports, then re-run the verb |
| `ErrPositionOutOfRange` | re-run with a card number in the range the message names |
| `ErrPositionFrozen` | re-run with a card number after the last begun card the message names |
| `ErrOnlyCard` | insert the replacement card first with `lyx loom insert-card`, then remove this one |
| `ErrSlugInvalid` | re-run with a slug of lowercase letters and digits joined by single hyphens |
| `ErrIntentInvalid` | re-run with a non-empty one-line `--intent` |
| `ErrNoRenumberJournal` | nothing to restore; insert or remove a card with `lyx loom insert-card` or `lyx loom remove-card` |
| `ErrRenumberJournalUndecodable`, `ErrRenumberJournalOutside` | delete the journal file the message names, then run `lyx loom validate-plan`, whose findings name any card left inconsistent |
| `ErrRestoreIncomplete` | re-run `lyx loom restore-renumber`; the journal is kept |
| any other error, the lease, `LoadState` and raw I/O failures | re-run the refused verb; nothing is mutated |

The verbs join `verbUsesLightweightWiring`, since they read only `c.env.AnchorPath` and webster's record and load no module config, and `resolveRunID` skips them as it skips `reject`, so `insert-card`'s and `remove-card`'s card number is never read as a run-id.
`Command` adds the three verbs to its `AddCommand` call.
The doc comments of `genericShedVerb` and `Arm`, which each enumerate loom's hand-written verbs (`start, validate-discussion, validate-plan, validate-description and approve`), already incomplete, are reworded to say "loom's hand-written verbs" without listing them, so the three new verbs do not make them wrong.

`refusal-spec.md`'s loom section gains one row per refusal above, triggers and classes as D2 assigns them (the journal-present, reversal-failure, plan, non-integer number, position, frozen, only-card, slug, intent, no-journal, undecodable and outside-path rows correctness halts; `restore-renumber: restore incomplete` transient), and its `raw I/O` row is widened to `stat`, `mkdir`, `read`, `write`, `rename` or `remove` of a status, seed, lock, park marker or records file, a plan card or overview file, or the renumber journal.
`loom-plan-spec.md`'s "Numbering and commit subject" section gains one sentence: inserting or removing a card renumbers every later card, and `lyx loom insert-card` and `lyx loom remove-card` do it.
`docs/overview.md`'s loom module line lists `insert-card|remove-card|restore-renumber` after `validate-plan`, and the verb paragraph gains one line for the three verbs.
`internal/loomcli/doc.go` gains a `# Card renumbering` section describing the three verbs, the lease and the journal recovery.

`renumber_test.go` (package `loomcli`, plans from `planFixture`-style `plankit.Write` fixtures under `c.env.AnchorPath`, with `plankit.WriteTree` for an orphan file and a planted journal) adds one table-driven test, `TestRenumberCmds`.
Cards 1 and 2 already pin each sentinel and the byte-identical directory, so its rows pin what the CLI adds: the exit code, the envelope and the exact way-forward clause `renumberRefusal` words, plus a byte-identical snapshot only where the CLI layer decides.

- Success rows: an `insert-card` and a `remove-card` at a middle position, envelope fields exact, including a `prose` entry for a planted `card 2` mention.
- Mapping rows, each asserting exit 1 and the exact way-forward clause: a planted journal, an overview with no Card Index, an orphan card file, an out-of-range number for each verb, a slug `a/b`, an empty `--intent`, a one-card plan's `remove-card 1`, `restore-renumber` with no journal and with a journal holding `{`.
- A non-integer number row, which `planparser` never sees, asserts its clause and a byte-identical directory.
- Frozen rows write a run record with `websterengine.SaveState` and refuse a position at the last begun card for a terminal batch, for a batch begun and not yet terminal, and for a record with no `Cards`, each asserting the clause and a byte-identical directory, and accept the position after it.
- A halt row sets `c.renumberFS` to fail a move and then the reversal, asserts the `restore-renumber` way forward, that `insert-card` and `remove-card` then refuse with the same way forward, and that `restore-renumber` then exits 0 and leaves the directory byte-identical to the pre-call snapshot.
- A raw-I/O row fails the journal's temporary write and asserts the grouped "nothing is mutated" message; a restore-incomplete row fails `restore-renumber`'s first write-back and asserts its re-run way forward.
- Lease rows, one per verb: `c.renumberFS` wraps `os` with a hook on every call that calls `lock.TryAcquireWriteLock` on `mutate.lock` in `websterengine.ScratchDir(anchorPath)` and records whether it was acquired.
  Each row asserts the hook ran at least once, through the journal's final `Remove`, and never acquired the lock, and that the same try acquires it after the verb returns, so a begin-batch waits on the lease through the last apply step.
  The `restore-renumber` row first leaves a journal through a halted `insert-card`, so its restore reaches the FS calls.
- An index row asserts `clihelp.RenderIndex(Command(), clihelp.AudienceOperator)` lists `insert-card`, `remove-card` and `restore-renumber`, the index the `ly:cli` skill loads.

`TestVerbUsesLightweightWiring` gains a `true` row for each of `insert-card`, `remove-card` and `restore-renumber`.
`TestResolveRunID` gains an `insert-card` row with `args: []string{"3", "x"}` and a `remove-card` row with `args: []string{"3"}`, each with `wantRunID: shedrun.SelfRunID`, as the `reject` row has, so a missing exception fails; `restore-renumber` takes no argument, and the existing no-argument rows already cover that path.
One new top-level test takes `internal/loomcli` past its row, so `test-budget.yaml` raises it by one.

This card adds three verbs that rename, rewrite and delete plan files: they can let through only a renumber of the current generation's top-level card files and Card Index, bounded by Card 1's and Card 2's refusals, the frozen begun cards and the lease.

**ImpactSummary:** new verbs in `internal/loomcli` plus one helper in `internal/websterengine`; the pre-run's run-id and wiring switches gain three names, and three docs and one spec gain rows or sentences, no existing behavior changes.

**Commit:** 3: loom: insert-card, remove-card and restore-renumber verbs
