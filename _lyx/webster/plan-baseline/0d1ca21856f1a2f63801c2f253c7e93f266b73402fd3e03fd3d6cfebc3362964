# Card 4 — loom-card-verbs

**Create:**
- `internal/loomcli/renumber.go`
- `internal/loomcli/renumber_test.go`
- `internal/loomcli#renumberDeps`
- `internal/loomcli#loomCLI.newRenumberDeps`
- `internal/loomcli#insertCardVerb`
- `internal/loomcli#removeCardVerb`
- `internal/loomcli#restoreRenumberVerb`
- `internal/loomcli#renumberRefusal`
- `internal/loomcli#renumberEnvelope`
- `internal/loomcli#loomCLI.insertCardCmd`
- `internal/loomcli#loomCLI.removeCardCmd`
- `internal/loomcli#loomCLI.restoreRenumberCmd`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#loomCLI.resolveRunID`
- `internal/loomcli/wiring_test.go`
- `internal/loomcli/arm_seed_test.go`
- `internal/loomcli/doc.go`
- `contracts/specs/refusal-spec.md`
- `contracts/specs/loom-plan-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/loomcli#approveVerb`
- `internal/loomcli#approveDeps`
- `internal/planparser#PlanDir`
- `internal/planparser#RenumberJournalPath`
- `internal/planparser#OSRenumberIO`
- `internal/planparser#InsertCard`
- `internal/planparser#RemoveCard`
- `internal/planparser#RestoreRenumber`
- `internal/websterengine#LoadState`
- `internal/websterengine#AcquireStateMutation`
- `internal/websterengine#Dir`
- `internal/websterengine#ScratchDir`
- `internal/websterengine#LastBegunCard`

**Intent:** Add the three `lyx loom` verbs, flat siblings of `validate-plan`, each a thin cobra shell over a verb function that takes its side effects as `renumberDeps`, the shape `approveVerb` and `approveDeps` set.

`renumberDeps` carries `planDir`, `journalPath`, `ops` (a `planparser.RenumberIO`), `loadState` (webster's run record, nil when none) and `acquireLease`, a `func() (release func() error, err error)` that takes webster's state-mutation lease and returns its release, so a test wrapper sees the release as well as the acquisition.
`newRenumberDeps` fills them from `c.env.AnchorPath`: `planparser.PlanDir`, `planparser.RenumberJournalPath`, `planparser.OSRenumberIO()`, `websterengine.LoadState` over `websterengine.Dir` and `websterengine.ScratchDir`, and `websterengine.AcquireStateMutation` over `websterengine.ScratchDir`, returning the acquired lock's `Release`.
All three verbs join `verbUsesLightweightWiring`, since they read only `c.env.AnchorPath`, and `TestVerbUsesLightweightWiring` gains their rows.
`resolveRunID` exempts `insert-card` and `remove-card` from its positional run-id read beside `reject`, since their first argument is a card position, never a run-id, so arming addresses the worktree's own run and never validates the position as a run-id; `TestResolveRunID` gains a row per exempted verb with a positional argument, each left addressing `shedrun.SelfRunID`.

Each verb acquires the lease first, before it reads the run record or the plan directory, and releases it after its last apply step, before it prints, so no begin-batch records a batch between the bound check and the renumber.
`insertCardVerb` and `removeCardVerb` then parse the raw `position` with `strconv.Atoi`, a non-integer taking the position refusal, load the run record, refuse a position at or before `websterengine.LastBegunCard`, naming the bound, and call `planparser.InsertCard` or `RemoveCard`; `restoreRenumberVerb` calls `planparser.RestoreRenumber`.
`insert-card` and `remove-card` are new edges that rename, rewrite and delete files; what bounds them: in the plan directory they touch only the current generation's card files and the Card Index of `00-overview.md` under `planparser.PlanDir`, never an archive, a prior generation, the overview's frontmatter or any other overview section; outside it they create the journal's parent directory, write the temporary journal, rename it to `planparser.RenumberJournalPath` and remove the journal, and the lease creates webster's scratch directory and its lock file; they refuse a plan that does not parse or whose index and files disagree, a position outside `first_card` to the last card (plus one for insert), a position at or before the last begun card, a batch begun and not yet terminal included, the only card's removal, a slug outside `planparser`'s slug rule and an empty or multi-line intent; they never commit, never approve and never set `approved:`.
`restore-renumber` is a new edge that rewrites and removes files; what bounds it: it holds the same lease; in the plan directory it writes and removes only the base names the journal lists, each a card file name or `00-overview.md` at the top level of `planparser.PlanDir`, and outside it removes only the journal, last; it refuses an absent journal, one that does not decode and one naming any other path, writing nothing; it never commits, never approves and never sets `approved:`.
What it lets through: it writes back whole pre-call files, all of `00-overview.md` included, and removes every journaled created name whatever that file holds by then, such as a stub body written after the interrupted insert, so it discards any edit made to the journaled files since the interrupted call, the only state that matches the run record the begun bound was checked against.

`renumberRefusal` is the one place the refusals are worded: it maps each `planparser` sentinel and the begun bound onto `loom: <verb>: <cause>; way forward: <step>`, so every message names its way forward and `planparser` never names a `lyx` verb.
The rows to add to `refusal-spec.md`'s loom section, each with its class and way forward: plan does not parse (correctness halt; fix the file the message names so `lyx loom validate-plan` parses it, then re-run the verb); Card Index and card files disagree (correctness halt; run `lyx loom validate-plan` and fix its `index-file-mismatch` and `card-numbering` findings, then re-run the verb); position out of range (correctness halt; pass a position from `first_card` to the last card, or one past it for `insert-card`); position at or before the last begun card (correctness halt; a begun card is frozen, so insert or remove after the card the message names); `remove-card` on the only card (correctness halt; insert the replacement card first, then remove); invalid slug (correctness halt; pass lowercase letters and digits joined by single hyphens); invalid intent (correctness halt; pass a non-empty one-line `--intent`); renumber journal present (correctness halt; run `lyx loom restore-renumber`, then re-run the verb); reversal failed (correctness halt; run `lyx loom restore-renumber`); `restore-renumber` with no journal (correctness halt; nothing to restore, `lyx loom insert-card` or `lyx loom remove-card` is the verb to run); journal does not decode or names a path outside the plan directory's top-level card and overview files (correctness halt; delete the named journal file, then run `lyx loom validate-plan`, whose findings name any card left inconsistent); restore failed partway (transient; re-run `lyx loom restore-renumber`, the journal is kept).
Widen the loom section's raw I/O row to name `rename`, the plan directory's card and overview files and the renumber journal, so an `ErrRenumberIOFailed`, a failed run-record read and a failed lease all join it with "re-run the refused verb; nothing is mutated".
`validate-plan`'s findings envelopes stay out of the table; these are refusals.

The success envelope, from `renumberEnvelope`: `plan_dir`, `moved` (one object per moved card with `old_number`, `new_number`, `old_file`, `new_file`), `created` or `removed` (the file), and `prose_mentions` (one object per line with `file`, `line`, `text`), which the agent checks by hand; `restore-renumber` reports `plan_dir`, `journal`, `restored` and `removed`.

Cobra: `insert-card <N> <slug> --intent <intent>` with `--intent` marked required, `remove-card <N>`, `restore-renumber`, all `clihelp.AudienceOperator` so `lyx help index`, which the `ly:cli` skill loads, lists them, each with a `Short` and a `Long` under the cap that names the renumber (file name prefix, heading number, explicit `Commit:` prefix, Card Index number), the heading-only stub the agent fills with Edit or Write, and the reported prose mentions; `Command` mounts them after `validatePlanCmd`.

Tests, in `renumber_test.go`, drive the verb functions over `plankit.Write` fixtures and `newRenumberDeps`' real path shape under a temp anchor, whose scratch directories do not exist yet, tier 1, and assert only what `loomcli` owns, the planparser behavior being pinned by cards 2 and 3: `TestRenumberVerbs`, a table over the three verbs (insert: exit 0 and the envelope's `moved`, `created` and `prose_mentions`; remove: exit 0 and the envelope's `removed`; restore after an injected reversal failure: exit 0 and the envelope's `restored` and `removed`), each row asserting the lease is held from the run-record read through every apply step: `acquireLease` wraps the real `websterengine.AcquireStateMutation` and wraps the returned release to record acquisition and release, and the wrapped `loadState` and the injected `ops` each fail the test at any call reached while the lease is not held, on the verb's own call path with no goroutine; `TestRenumberVerbs_Refusals`, a table over every row above for each verb, a non-integer position included, each asserting exit 1 and the message's way forward, plus: the begun bound reached with a done batch and with a batch begun and not yet terminal, each also asserting the plan directory byte-identical, since only `loomcli` reaches it; an injected reversal failure reaching the correctness-halt message naming `lyx loom restore-renumber`, after which both other verbs refuse naming the same way forward; and the no-journal, invalid-journal and partial-failure restore refusals.
`internal/loomcli` is at its test budget, so raise its row by two.

Docs in this commit: `internal/loomcli/doc.go` gains a "Plan card verbs" section describing the three verbs, their bounds, the journal and the envelope, and that `restore-renumber` discards any edit made to the journaled files since the interrupted call; `docs/overview.md`'s loom verb list and verb paragraph name them; `contracts/specs/loom-plan-spec.md`'s "Numbering and commit subject" section says a card is inserted or removed with `lyx loom insert-card` or `lyx loom remove-card`, which move every later card by one in file name prefix, heading number, explicit `Commit:` prefix and Card Index number, and report prose mentions of a moved number without rewriting them.

**ImpactSummary:** Three new verbs mounted on `lyx loom` and three names added to the lightweight-wiring set; `resolveRunID` gains two exempted verbs beside `reject`, so no existing verb's arming or behavior changes, and the refusal table and docs gain rows and paragraphs only.
