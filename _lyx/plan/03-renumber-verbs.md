# Card 3 — renumber-verbs

**Create:**
- `plan:internal/loomcli#loomCLI.insertCardCmd` -> `func (c *loomCLI) insertCardCmd() *cobra.Command`
- `plan:internal/loomcli#loomCLI.removeCardCmd` -> `func (c *loomCLI) removeCardCmd() *cobra.Command`
- `plan:internal/loomcli#loomCLI.restoreRenumberCmd` -> `func (c *loomCLI) restoreRenumberCmd() *cobra.Command`
- `plan:internal/loomcli#renumberWayForward` -> `func renumberWayForward(err error, journalPath string) string`
- `internal/loomcli/renumber_test.go`
- `plan:internal/loomcli#TestRenumberCmds` -> `func TestRenumberCmds(t *testing.T)`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#loomCLI`
- `internal/loomcli#newLoomCLI`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#loomCLI.wireLightweight`
- `internal/loomcli#loomCLI.resolveRunID`
- `internal/loomcli#TestVerbUsesLightweightWiring`
- `internal/loomcli#TestResolveRunID`
- `internal/loomcli#TestNewLoomCLI_SetsInjectedSeams`
- `internal/loomcli/doc.go`
- `contracts/specs/refusal-spec.md`
- `contracts/specs/loom-plan-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/planparser#InsertCard`
- `plan:internal/planparser#RemoveCard`
- `plan:internal/planparser#RestoreRenumber`
- `plan:internal/planparser#RenumberJournal`
- `plan:internal/planparser#RenumberFS`
- `plan:internal/planparser#OSRenumberFS`
- `plan:internal/testkit/plankit#FaultFS`
- `plan:internal/websterengine#LastBegunCard`
- `internal/websterengine#AcquireStateMutation`
- `internal/websterengine#LoadState`

**Intent:**
Add the three `lyx loom` verbs in a new `internal/loomcli/renumber.go`, beside `validate-plan`, and mount them in `Command`'s `AddCommand` list (D1, D3).
Each is a flat leaf with `Short` one lower-case line, `audience` `operator` like `validate-plan`, a `Long` with an example, and JSON envelopes through `internal/output`, checking `clihelp.ShouldAbort` first (`PATTERN-cli-cobra`):
- `insert-card <position> <slug> --intent <intent>`: `cobra.ExactArgs(2)`, `--intent` marked required.
- `remove-card <position>`: `cobra.ExactArgs(1)`.
The argument tokens are lower-case, since `cmd/lyx/clitree_test.go`'s `Use` grammar admits `<[a-z0-9-]+>` only.
- `restore-renumber`: `cobra.NoArgs`.

Each RunE acquires `websterengine.AcquireStateMutation(websterengine.ScratchDir(c.env.AnchorPath))` first and releases it after the planparser call returns, so the lease covers the run-record read through the last apply step and a begin-batch blocks on it meanwhile (D2).
`insert-card` and `remove-card` then load `websterengine.LoadState(websterengine.Dir(anchor), websterengine.ScratchDir(anchor))` and pass `websterengine.LastBegunCard(st)` as `frozenThrough`; with no run record it is 0 and every position is open.
They call `planparser.InsertCard` or `planparser.RemoveCard` with `c.renumberFS`, `planparser.PlanDir(anchor)` and `planparser.RenumberJournal(anchor)`; `restore-renumber` calls `planparser.RestoreRenumber` the same way.
A `<position>` that does not parse as an integer is refused before the lease as a position out of range, its reaching input being `remove-card x`.

A success prints `output.Ok` with `plan_dir`, `moved` (each `from`, `to`, `from_file`, `to_file`), `created` or `removed`, and `prose` (each `file`, `line`, `text`) for the two renumber verbs, and `restored` (the file names written back) for `restore-renumber`.
A refusal prints `output.Err` with the planparser error's text and a trailing `way forward: ` clause from `renumberWayForward`, which maps each sentinel with `errors.Is`:
- `ErrRenumberJournalPresent`, `ErrRenumberReversalFailed`: run `lyx loom restore-renumber`.
- `ErrRenumberPlanInconsistent`: fix the plan until `lyx loom validate-plan` reports no `index-file-mismatch` or `card-numbering` finding, then re-run the verb.
- `ErrRenumberPosition`: pass a position in the range the message names.
- `ErrRenumberFrozen`: pass a position after the last card of the last begun batch, which the message names.
- `ErrRenumberOnlyCard`: insert the replacement card with `lyx loom insert-card` first, then remove this one.
- `ErrRenumberSlug`: pass a slug of lowercase ASCII letters and digits joined by single hyphens.
- `ErrRenumberIntent`: pass a non-empty one-line `--intent`.
- `ErrRenumberNoJournal`: nothing to restore; insert or remove a card with `lyx loom insert-card` or `lyx loom remove-card`.
- `ErrRenumberJournalInvalid`: delete the journal file at `journalPath` and run `lyx loom validate-plan`, whose findings name any card left inconsistent.
- `ErrRestorePartial`: re-run `lyx loom restore-renumber`; the journal is kept.
- `ErrRenumberIO`, a lease failure and a run record that cannot be loaded: re-run the refused verb; nothing is mutated.
No clause names a denied command (`PATTERN-no-denied-recovery`).

Bounds of the new edge: the verbs rename, rewrite and delete only top-level card files and the Card Index of `00-overview.md` in the current plan directory, plus the journal under `.lyx`; they refuse an inconsistent plan, a frozen or out-of-range position, a bad slug or intent and a pending journal before any change, never commit, approve or set `approved:`, and the row's commit seam commits the directory as today.
`restore-renumber` writes only the names its journal lists and refuses any other name, so its reach is the same files; its cost is that it discards edits made to those files since the interrupted call.

Add a `renumberFS planparser.RenumberFS` field to `loomCLI`, which `newLoomCLI` sets to `planparser.OSRenumberFS()`, and assert it non-nil in `TestNewLoomCLI_SetsInjectedSeams`, adding `renumberFS` to the fields its `//testtiming:keep` reason names.
Add the three verbs to `verbUsesLightweightWiring`, since they read only `c.env.AnchorPath` and load no module config, and update its doc comment's set and `wireLightweight`'s doc comment, which lists the verbs on the lightweight path and why; add three `true` rows to `TestVerbUsesLightweightWiring`.
`resolveRunID` reads `args[0]` as the run-id for every verb but `reject`; extend that exception to `insert-card` and `remove-card`, whose positional arguments are a card position and slug, reword its comment that names `reject` as the one such verb, and add rows to `TestResolveRunID` asserting `insert-card 3 x` and `remove-card 3` leave `runID` at `shedrun.SelfRunID`.

`TestRenumberCmds`, a new tier-1 table test in a new `internal/loomcli/renumber_test.go`, drives each leaf command built on a hand-populated `*loomCLI` (as `validate_test.go` does), over plans written with `plankit.Write` under `t.TempDir()`:
- insert at a middle position and remove at a middle position: exit 0, the envelope's `moved`, `created`/`removed` and `prose` keys as expected.
- a run record saved with `websterengine.SaveState` holding one non-terminal batch whose `Cards` is `["02-b"]`: `insert-card 2` is refused naming card 2 and the frozen way forward, and `insert-card 3` succeeds.
- one row per way-forward clause above, each reached through its real input: an orphan card file, `remove-card 9`, `remove-card x`, the only card, slug `a/b`, intent `a\nb`, a `plankit.FaultFS` failing the Card Index write (grouped row), a `FaultFS` failing a move and the reversal's first write (restore-renumber), then `insert-card` over the left journal, then `restore-renumber` with no journal, over a journal of non-JSON bytes, and through a `FaultFS` failing its second write; each asserts exit 1 and the clause only, since card 2's tests pin the plan directory's bytes and the restore.
- the lease, one row per verb (`insert-card`, `remove-card`, and `restore-renumber` over a journal a reversal failure left): a `FaultFS` whose `OnWrite` hook probes `lock.TryAcquireWriteLock` on the literal file name `mutate.lock` in `websterengine.ScratchDir(anchor)` (the name is unexported in `websterengine`, as `TestAcquireStateMutation_ExcludesSecondHolder` spells it) records whether it was held during the apply, releasing the lock at once if the probe unexpectedly acquires it so the verb's own release cannot block, and a probe after the verb returns acquires it.
Raise `internal/loomcli`'s row in `cmd/lyx/testdata/test-budget.yaml` from 195 to 196.

Docs in the same commit:
- `contracts/specs/refusal-spec.md`'s loom section gains one row per refusal above but the grouped one, classed: journal present, reversal failed, plan inconsistent, position out of range, position frozen, only card, invalid slug, invalid intent, journal undecodable or naming a path outside the plan directory's top-level card and overview files, and no journal are correctness halts; restore partial is transient; each Way forward cell quotes its clause.
  The `raw I/O` row's trigger becomes `stat`, `mkdir`, `read`, `write`, `rename` or `remove` of a status, seed, lock, park marker or records file, of the plan directory's card or overview files, or of the renumber journal.
- `contracts/specs/loom-plan-spec.md`'s "Numbering and commit subject" section gains: a card is inserted or removed only with `lyx loom insert-card` or `lyx loom remove-card`, which renumber the later cards' file prefix, heading, `Commit:` prefix and Card Index number, and `lyx loom restore-renumber` restores a plan an interrupted call left.
- `internal/loomcli/doc.go` gains a `# Renumber` section describing the three verbs, their bounds, the journal and the lease.
- `docs/overview.md`'s loom module line, which lists every `lyx loom` verb, adds the three, and the verb paragraph after `validate-plan --rework` gains one sentence each.
The lyx command index the `ly:cli` skill loads is `lyx help index`, generated from each command's `Short` and audience, so the three `Short` lines are its entries.

**ImpactSummary:** Three new operator verbs under `lyx loom`, one new loomCLI seam field, the lightweight-wiring and run-id sets each extended by the new verb names only; existing verbs' behavior is unchanged, and four doc files and one budget row move with them.
