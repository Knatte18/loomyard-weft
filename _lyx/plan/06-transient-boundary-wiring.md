# Card 6 — wire the classifier at the producer and bootstrap boundaries

**Edit:**
- `internal/shedbuild/newshed.go`
- `internal/shedbuild/newshed_test.go`
- `internal/shedbuild/seam_enforcement_test.go`
- `internal/loomcli/arm.go`
- `internal/battencli/arm.go`

**Create:**
- `internal/loomcli/integration_transientbootstrap_test.go`

**Prosa:**
- `CONSTRAINTS.md`

**Uses:**
- `plan:internal/shedtransient#Class`
- `plan:internal/shedtransient#Mark`

**Intent:**
Turn the mark on for real (decision record D1, D2, D12): every recipe shed classifies its producer errors, and every step bootstrap classifies its own failures, through the one `internal/shedtransient` mapping.

Producer boundary: `shedbuild.NewShed` sets the assembled `Shed`'s `Transient` field to `shedtransient.Class`.
`NewShed` is the single assembler both recipe sheds (`loomrecipe.New`, `battenrecipe`) go through, so no recipe package changes.
`shedbuild`'s import allowlist (`seam_enforcement_test.go`) admits `internal/shedtransient`, with a one-line reason beside the entry.

Bootstrap boundary: `loomPreStep` and `battenPreStep` pass each error they return through `shedtransient.Mark`, leaving every kind unchanged.
The busy refusal is never transient by construction, since `Mark` leaves an unclassified error untouched.
In loom, the seed-and-commit stage pushes through fabric, so a push that cannot reach the remote now reaches the envelope as `kind: bootstrap, transient: git-transport`.

`CONSTRAINTS.md` gains a **Transient Stop Invariant** section after the Shed Verb-Set Invariant, stating:
- the mark is declared only in `internal/shedengine`, and its class set is closed at the declaration (the set is named there, not restated here);
- a lower-level package exposes its own classification, and `internal/shedtransient` alone translates it into the mark;
- the mark is set only at a producer boundary (the `Shed.Transient` classifier `shedbuild.NewShed` tells) or a step bootstrap boundary (a `PreStep` hook);
- a gate verdict (`blocked`, `awaiting`, `paused`) never carries it;
- rationale: a mark added at a new site changes what the ly-drive driver re-steps without a human deciding.

Tests:
- `newshed_test.go`: the assembled shed's `Transient` is non-nil and classifies an error wrapping `shuttleengine.ErrNotStarted` as `agent-start` and a plain error as empty.
- `integration_transientbootstrap_test.go` (`//go:build integration`, under the package's existing hermetic `TestMain`): build a hub with `hubforge`, add a pair, point the weft side's push URL at `http://127.0.0.1:1/unreachable.git`, and run `lyx loom step` through `RunCLIIn` in the warp worktree.
  The envelope is an error with `kind: bootstrap` and `transient: git-transport`.
  A second case points the push URL at a local bare repository whose `pre-receive` hook rejects every push, and asserts `kind: bootstrap` with an empty `transient`.

**ImpactSummary:** Every recipe shed and both step bootstraps now classify their errors; only the persisted `transient` field and the envelope key change, never a kind, a state or an error text.

**Commit:** 6: shed: classify producer and bootstrap failures as transient
