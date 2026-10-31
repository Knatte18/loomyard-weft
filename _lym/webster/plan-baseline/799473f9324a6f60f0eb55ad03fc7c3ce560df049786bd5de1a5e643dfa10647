# Card 15 — darn-write-scope-guards

**Create:**
- `internal/shedrun#StopMarker`

**Edit:**
- `internal/loomshed#DarnDeps`
- `internal/loomshed#darnWrite.Call`
- `internal/loomshed#TestDarnWrite_Call`
- `internal/loomshed#darnWrite.checkSeams`
- `internal/loomshed#darnFixture`
- `internal/loomshed#newDarnFixture`
- `internal/loomshed#darnFixture.session`
- `internal/loomshed#darnSession.Call`
- `internal/loomcli#intentionallyNil`
- `internal/battencli#intentionallyNil`
- `internal/loomshed#TestGateRows_StopsNameAWayForward`
- `internal/loomshed/doc.go`
- `internal/shedrecipe#darnWriteEntry`
- `internal/shedrecipe#TestDarnWriteEntry_ConstructionFailures`
- `internal/shedrecipe#newTestEnv`
- `internal/testkit/envkit#FullEnv`
- `internal/darnengine#DarnInputs`
- `internal/darnengine#DarnTable`
- `internal/darnengine#TestDarnSpec`
- `internal/darnengine#TestDarnSpec_Refusals`
- `internal/darnengine/doc.go`
- `internal/loomcli#darnDeps`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_DarnRecipe`
- `contracts/stencils/darn/darn-template-write.md`
- `contracts/recipes/darn-recipe.yaml`
- `docs/overview.md`

**Uses:**
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/shedrecipe/doc.go`
- `internal/shedrun/doc.go`

**Intent:** Darn-Write reads the record Record-Write wrote as its scope, refuses a missing one before spawning, and halts on its writer's stop marker (D4, D6), and darn's Drift rows read the record (D4).

Darn-Write before its session:
- `DarnDeps` gains `RecordPath string`, `SupportLogPath string`, `CommittedRecord func() (bool, error)` (whether the `_lym` weft repository's HEAD holds the record) and `StopMarkerPath string`; `darnWriteEntry` requires the three paths absolute and the seam non-nil, and `darnWrite.checkSeams` names a nil `CommittedRecord` or an empty path, so a hand-built receiver must fill all four.
- `darnWrite.Call` first stats `RecordPath`. A missing record halts Stuck before any session, with one way forward whatever the task branch holds: `run "lym loom goto --to Record-Write", then "lym loom resume"`. When `CommittedRecord` answers true, the working copy was removed after Record-Write ran, and a record written again from the board entry would drop its committed `Added after Discussion` entries, so the way forward is `restore the decision record from the _lym weft repository's HEAD with "git -C _lym checkout HEAD -- discussion/decision-record.md", then run "lym loom resume"`. A record present beside a missing support log halts the same way before any session, naming `run "lym loom goto --to Record-Write", then "lym loom resume"`, since Record-Write writes only the missing support log and keeps the record byte for byte. That route covers a run in flight when this lands: parked on Drift-Baseline, Drift-Await or Darn-Write with no record, it is moved with the goto; Drift-Baseline then passes again keeping its earlier scope record and capture on a branch with no commit past the branch point, and on a branch with commits it takes no capture (driftengine's doc), so the resume reaches Darn-Write.
- It then removes the stop marker at `StopMarkerPath` before it spawns each session.

After the session, a Done handoff and a gate-failed Stuck alike, and ahead of card 13's guard, `darnWrite.Call` reads the stop marker. A marker present halts the run Stuck, its reason the change description's text (the stop report), read from the session's output pointer `Path`, the description the writer's one output file holds, followed by `way forward: record the answer with "lym loom decision add", then run "lym loom resume"`. So a stop report the `verify` or `description` gate refuses still halts with the description as its reason, and the halt lets nothing through: the run returns Stuck before it reaches Drift-Verify, whatever earlier spawns committed. A task whose right outcome is no code change halts the same way and its parent settles it.

`shedrun.StopMarker` names the marker in the run's directory, `shedrun.RunDir`, as D6 places it; the status commit may take a present marker, and the removal before the next spawn deletes it again. `darnDeps` wires the record path, the stop marker and `CommittedRecord` over `fabricengine.CommittedAnchoredFile` of `DiscussionDecisionRecordRel`.

The writer stencil and its table:
- `DarnInputs` drops `SupportLogPath` and gains `StopMarkerPath`; `DarnTable` renders the `stop_marker_path` marker and no `support_log_path`.
- `darn-template-write.md` drops its Step 3: the writer never writes or edits the record or the support log, and reads the record as its scope, the board entry's body under `## Goal` and the `Added after Discussion` entries over it, which win where they differ.
- Before any code it still checks whether landed work or another entry or run overtook or contradicts the entry, reading the board, its touching entries, `Decision for <slug>:` notes and `lym loom peers`. With a parent it asks the parent to record the answer with `lym loom decision add` and follows the resulting entry. With no parent, a `Decision for <slug>:` note never settles the stop: an outdated entry with no `Added after Discussion` entry answering it stops the writer again, whatever note answers it. The writer then writes the change description as a stop report in the final-summary spec's form, naming `lym loom decision add` then `lym loom resume` as the way forward and never a board note, writes the stop marker at `{{.stop_marker_path}}`, and ends its turn with no commit.
- The writer applies each bullet of the record's `## PATTERN changes` section to `PATTERN.md` in its own commits (D3).
- The stencil points at the discussion spec and restates none of it (PATTERN-stencil-ownership); its "after you hand off" section drops the discussion check.

Darn's Drift rows: `darn-recipe.yaml`'s Drift-Baseline and Drift-Verify take `subject: decision-record`, as tapestry's do, and `wire`'s `driftSubject` branch goes, every loom run telling the Drift agents the record. The recipe header says the writer reads the record Record-Write wrote and never writes it.

Tests:
- `TestDarnWrite_Call` runs on a fixture rebuilt over a temporary directory: `newDarnFixture` writes a record and a support log there and fills `RecordPath`, `SupportLogPath`, `StopMarkerPath`, `CommittedRecord` (false) and card 13's `CheckRecord`, so the existing rows still reach the session, and `darnFixture.session` gains a marker switch making `darnSession.Call` write the stop marker. A missing record halts before the session factory is called, naming the goto and resume; with `CommittedRecord` true it names the restore from HEAD instead; a missing support log beside a present record halts naming the goto and resume.
- A stop marker written by the fake session halts with the description text after a Done handoff and after a gate-failed Stuck, on a first spawn and on a respawn whose earlier spawn committed work, naming `lym loom decision add` then `lym loom resume`; a resume whose session writes the marker again, as a no-parent writer does while no `Added after Discussion` entry answers its stop, halts again. That a recorded answer reaches the perch is card 12's `TestDarnPerch_SharesLoomsPerchBarTheAnswerKey`, the record being the perch's answer key, and reaching the resumed writer is the stencil's reading of the record.
- A stale marker present before the spawn is removed and does not halt.
- `TestGateRows_StopsNameAWayForward` covers both new reasons.
- `TestDarnSpec` asserts the `stop_marker_path` value; `TestDarnSpec_Refusals` refuses an empty `StopMarkerPath`.
- `TestDarnWriteEntry_ConstructionFailures` refuses a relative `RecordPath`, `SupportLogPath` or `StopMarkerPath` and a nil `CommittedRecord`.
- `intentionallyNil` in loomcli and battencli lists `Darn.CommittedRecord`, which only a darn seed fills.
- `TestWire_DarnRecipe` asserts the wired paths and the decision-record drift subject.
`docs/overview.md`'s darn paragraph follows.

**ImpactSummary:** Darn-Write gains a pre-spawn record refusal and a post-session stop-marker halt, its stencil stops writing the record, and darn's drift subject becomes the record; tapestry is untouched.
