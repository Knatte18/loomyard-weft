# Card 4 — loom-renumber-verbs

**Create:**
- `plan:internal/loomcli#loomCLI.insertCardCmd` -> `func (c *loomCLI) insertCardCmd() *cobra.Command`
- `plan:internal/loomcli#loomCLI.removeCardCmd` -> `func (c *loomCLI) removeCardCmd() *cobra.Command`
- `plan:internal/loomcli#loomCLI.restoreRenumberCmd` -> `func (c *loomCLI) restoreRenumberCmd() *cobra.Command`
- `plan:internal/loomcli#renumberRefusal` -> `func renumberRefusal(verb string, err error) string`
- `plan:internal/loomcli#TestRenumberCmds` -> `func TestRenumberCmds(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#loomCLI.resolveRunID`
- `internal/loomcli#TestVerbUsesLightweightWiring`
- `internal/loomcli#TestResolveRunID`
- `internal/loomcli/doc.go`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/planparser#InsertCard`
- `plan:internal/planparser#RemoveCard`
- `plan:internal/planparser#RestoreRenumber`
- `plan:internal/planparser#RenumberJournal`
- `plan:internal/planparser#RenumberOps`
- `plan:internal/websterengine#LastBegunCard`
- `internal/websterengine#AcquireStateMutation`
- `internal/websterengine#LoadState`
- `internal/planparser#PlanDir`

**Intent:**
Add the three flat `lyx loom` verbs (D1, D2, D3), in a new file `internal/loomcli/renumber.go`, registered in `Command` beside `validate-plan`, each with `Short`, `Long` with examples, the `clihelp.AudienceOperator` audience, as `validate-plan` carries, and JSON envelopes (`PATTERN-cli-cobra`).
The operator audience is what puts them in the command index the `ly:cli` skill loads, since `lyx help index` renders the operator audience by default (`internal/helpcli/cli.go`) and `clihelp.RenderIndex` lists only the requested audience:

- `insert-card <N> <slug> --intent "<one-line intent>"`, `cobra.ExactArgs(2)`, `--intent` required.
- `remove-card <N>`, `cobra.ExactArgs(1)`.
- `restore-renumber`, `cobra.NoArgs`.

`verbUsesLightweightWiring` admits all three, since they read only `c.env.AnchorPath` and load no module config, and its doc comment, which lists the set, names them as the card-renumbering verbs.
`loomCLI.resolveRunID` leaves the run-id at `self` for `insert-card` and `remove-card`, whose positionals are a card number and a slug, never a run-id, as it already does for `reject`; its doc comment and the `arm.go` file header, which both say the run-id is `args[0]` when present, name the exception.
`loomCLI` gains a `renumberOps planparser.RenumberOps` field, zero in production, through which a test injects a file-operation failure.

Each verb resolves `planDir` with `planparser.PlanDir(c.env.AnchorPath)` and the journal with `planparser.RenumberJournal(c.env.AnchorPath)`, then acquires `websterengine.AcquireStateMutation(websterengine.ScratchDir(anchor))` and holds it until the last apply step has returned, so a begin-batch, which takes the same lease, cannot record a batch between the bound check and the renumber.
`insert-card` and `remove-card` then load webster's run record with `websterengine.LoadState(websterengine.Dir(anchor), websterengine.ScratchDir(anchor))` and refuse, before calling planparser, a position `N` at or before `websterengine.LastBegunCard(st)` when that is above 0: "card N is at or before card L, the last card of the last begun webster batch, and a begun card is frozen; way forward: insert or remove a card numbered after L".
A non-numeric or non-positive `N` is refused as "N is not a card number; way forward: pass the card's number, as the Card Index lists it".
They then call `planparser.InsertCard` or `planparser.RemoveCard` and print an ok envelope with `moved` (each move's `from`, `to`, `old_file`, `new_file`), `created` or `removed`, and `prose_mentions` (each `file`, `line`, `text`), for the agent to check by hand.
They never commit, never approve and never write `approved:`.

`renumberRefusal` renders a refusal message naming its way forward from the sentinel `errors.Is` finds:
- `ErrRenumberJournalPresent` and `ErrRenumberReversalFailed`: run `lyx loom restore-renumber`, then re-run the verb.
- `ErrPlanUnparseable`: `lyx loom validate-plan` names what does not parse; fix it, then re-run the verb.
- `ErrPlanInconsistent`: fix the `index-file-mismatch` and `card-numbering` findings `lyx loom validate-plan` names, then re-run the verb.
- `ErrCardPositionOutOfRange`: name a position in the range the message states.
- `ErrOnlyCard`: insert the replacement card first with `lyx loom insert-card`, then remove this one.
- `ErrInvalidCardSlug`: pass a slug of lowercase letters and digits joined by single hyphens.
- `ErrInvalidCardIntent`: pass a non-empty, one-line `--intent`.
- `ErrRenumberNoJournal`: nothing to restore; insert or remove a card with `lyx loom insert-card` or `lyx loom remove-card`.
- `ErrRenumberJournalCorrupt` and `ErrRenumberJournalPathOutside`: delete the journal file the message names, then run `lyx loom validate-plan`, whose findings name any card left inconsistent.
- `ErrRenumberRestorePartial`: re-run `lyx loom restore-renumber`; the journal is kept.
- any other error, a raw I/O failure: re-run the refused verb; nothing is mutated.

None of these names a denied form (`PATTERN-no-denied-recovery`).
The bound on the new edges is D2's: only top-level card files and the Card Index of `00-overview.md` in `planparser.PlanDir`, never a begun card, never a plan that fails to parse or is inconsistent, never while a journal exists, and restore writes only validated top-level base names; they let through any position after the last begun card, and restore discards edits made since the interrupted call to the files the journal names.

`contracts/specs/refusal-spec.md`'s loom section gains one row per refusal above, each with its trigger, class and way forward: the journal-present, unparseable, inconsistent, out-of-range, begun-card, bad card number, only-card, slug and intent rows and the restore journal-corrupt and path-outside rows as correctness halts, the reversal-failed row as its own correctness halt, the restore no-journal row as a correctness halt, and the restore-partial row as a transient whose way forward is re-running `lyx loom restore-renumber`.
The loom section's raw I/O row widens to name `rename` beside `stat`, `mkdir`, `read`, `write` and `remove`, and the plan directory's card and overview files and the renumber journal beside the files it already names.

`TestRenumberCmds`, in a new file `internal/loomcli/renumber_test.go`, drives each verb's leaf command on a hand-populated `*loomCLI` (`env.AnchorPath`, plan written with `plankit` under `planparser.PlanDir`) through `clihelp.Execute`, and in rows asserts the exit code and the envelope; the plan directory's bytes are asserted only on the refusals loomcli owns, since card 1's and card 2's rows pin them for every planparser sentinel:
- insert at a middle position: the envelope's `moved`, `created` and `prose_mentions` match the disk;
- remove at a middle position: `moved` and `removed`;
- begun-card refusal with a run record whose last batch is begun and not terminal (written with `websterengine.SaveState`), at the last begun card for both verbs, the plan directory byte-identical, and an insert one past it accepted;
- one row per planparser sentinel above, reached through the verb (journal present, an overview with no `## Card Index` heading, an orphan card file, an insert at the last card plus two, the only card, a slug `a/b`, an `--intent` holding a newline, restore with no journal, restore with a corrupt journal, restore with a journal naming `../x.md` in card 1's JSON shape), asserting the envelope message holds the way-forward text;
- a raw failure injected through `renumberOps` at a move, asserting the re-run way forward;
- a reversal failure injected through `renumberOps`, asserting the `lyx loom restore-renumber` way forward, then `insert-card` and `remove-card` refusing with the same way forward, then `restore-renumber` succeeding;
- a write-back failure injected through `renumberOps` into `restore-renumber` over a journal a reversal failure left, asserting the transient way forward "re-run `lyx loom restore-renumber`; the journal is kept" and the journal still on disk;
- `N` given as `x`, the plan directory byte-identical;
- the state-mutation lease, one row per verb (`insert-card`, `remove-card`, `restore-renumber` over a left journal): a `renumberOps.Remove` hook that, when called on the journal path (each verb's last step), calls `lock.TryAcquireWriteLock` on `mutate.lock` under `websterengine.ScratchDir(anchor)` and records the result before removing; the row asserts the hook saw the lease held, and a `TryAcquireWriteLock` after the verb returns succeeds, so a begin-batch started meanwhile waits until the verb releases it;
- the command index: `clihelp.RenderIndex(Command(), clihelp.AudienceOperator)` names `insert-card`, `remove-card` and `restore-renumber`.

`TestVerbUsesLightweightWiring` gains rows `insert-card`, `remove-card` and `restore-renumber` wanting true; `TestResolveRunID` gains rows `insert-card` with args `3 new-card` and `remove-card` with arg `3`, each never refusing and leaving the run-id at `self`.
`internal/loomcli/doc.go` gains a `# Card renumbering` section describing the three verbs, the lease, the begun-card bound, the journal and the envelope; `docs/overview.md`'s loom verb list and its verb paragraphs name the three verbs; the command index the `ly:cli` skill loads renders from the verbs' operator audience and `Short` text with no file to edit.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/loomcli` by one.

**ImpactSummary:** three new loom verbs registered in `Command`; `verbUsesLightweightWiring` and `resolveRunID` each gain verb names without changing any existing verb's behavior, and `loomCLI` gains one zero-valued field.
