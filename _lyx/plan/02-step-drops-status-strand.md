# Card 2 — step brings reed up and does no status-strand work

**Edit:**
- `internal/loomcli/arm.go`
- `internal/loomcli/sharedbootstrap.go`
- `internal/shedcli/parity_test.go`

**Create:**
- `internal/loomcli/smoke_drivingsurface_test.go`

**Uses:**
- `internal/loomcli#seedGoDriverRun`
- `internal/loomcli#seedLLMDriver`
- `internal/loomcli#statusStrandDisplayName`

**Intent:**
`lyx shed step` is what the ly-drive driver runs on every step, so while `loomPreStep` calls `ensureStatusStrand` it re-adds the status strand on every llm step.
Make `step` driver-blind on the substrate: it brings reed up and touches no strand.

In `arm.go`'s `loomPreStep`, replace the `c.ensureStatusStrand()` call with `c.reed.Up()`, discarding the `UpResult`.
Keep everything around it unchanged: the early run-lock probe stays above seeding and reed, the bootstrap lock is still acquired before and released right after, a failed `Up` still releases the lock and returns `shedverbs.KindBootstrap`, and the driver `seedAndCommitBootstrap` returns stays discarded.
`Up` stays because the producers under `step` spawn agents into reed panes.
Reword `loomPreStep`'s doc comment ("then the status-strand ensure") and the early-probe comment ("churn the status strand") so they name reed `Up` instead;
the doc comment also says `step` never adds or removes the status strand — removal needs the driver, which `step` must not read, and an unconditional removal would strip the band from a halted go run an operator steps by hand.

In `sharedbootstrap.go`, reword the file header: `step` now wraps reed `Up` alone in its bootstrap-lock window, not the strand block, and `ensureStatusStrand` is `start`'s alone.
`ensureStatusStrand`'s body is untouched by this card.

In `internal/shedcli/parity_test.go`, correct the two comments that place `step`'s busy refusal "above seedAndCommitBootstrap and ensureStatusStrand" so they say above `seedAndCommitBootstrap` and reed `Up`.

Create `smoke_drivingsurface_test.go` (build tag `smoke`, package `loomcli`, with a file doc comment in the style of its siblings: zero real LLM subprocesses, fixtures reused from `smoke_test.go` and `smoke_driverstrand_test.go`), holding:
- `TestSmokeStep_AddsNoStatusStrandOnEitherDriver`: table over a go-seeded run (`seedGoDriverRun`) and an llm-seeded run (`seedLLMDriver`), each crossed with `lyx loom step` and `lyx shed step`, each case on its own `newWiredPairFixture` with `registerBootstrapTeardown`;
  after the step, `probeReedEngine(...).Status()` succeeds (reed is up) and `statusStrandCount` for `statusStrandDisplayName` is 0.
- `TestSmokeStep_LeavesAPreexistingStatusStrandUntouched`: on a go-seeded fixture, bring reed up through the probe engine and add one strand with `statusStrandAddSpec` over a long-lived shell command, then run `lyx loom step`;
  afterwards exactly one strand carries `statusStrandDisplayName` and its guid is the one added before the step.

**ImpactSummary:** `lyx loom step` and `lyx shed step` on loom stop keeping, replacing or adding the status strand; `start` is unchanged, and the parity tests' refusal arm never reached reed, so they pass unchanged apart from comments.

**Commit:** 2: loomcli: step brings reed up and does no status-strand work
