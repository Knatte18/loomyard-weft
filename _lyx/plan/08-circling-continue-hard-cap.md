# Card 8 — circling-continue-hard-cap

**Edit:**
- `internal/loomcli#circlingWayNot`
- `internal/loomcli#circlingDeps`
- `internal/loomcli#circlingVerb`
- `internal/loomcli#circlingState.preRun`
- `internal/loomcli#loomCLI.circlingCmd`
- `internal/loomcli/doc.go`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/loomcli#circlingFake`
- `internal/loomcli#circlingFake.deps`
- `internal/loomcli#TestCirclingVerb_Records`
- `internal/loomcli#TestCirclingVerb_Refusals`
- `internal/shedadapters#TestRecordCirclingDecision_WritesPendingFile`

**Create:**
- `plan:internal/shedadapters#LatestRound` -> `func LatestRound(runDir string) (int, error)`
- `plan:internal/loomcli#circlingWayHardCap` -> `const circlingWayHardCap = "way forward: fix the file or key the message names, then re-run the verb"`
- `plan:internal/loomcli#perchHardCap` -> `func perchHardCap(location *lyxcwd.Location, row string) (int, string, error)`
- `plan:internal/loomcli#TestPerchHardCap` -> `func TestPerchHardCap(t *testing.T)`

**Uses:**
- `internal/loomengine#LoadConfig`
- `internal/darnengine#LoadConfig`
- `internal/loomcli#seedDarnHubConfig`
- `internal/loomcli#hubLocation`

**Intent:** `lyx loom circling continue` refuses on a round at or past its perch's hard cap, for every caller, the arbiter included (D5), so neither a parent nor an arbiter can grant round H+1.
This adds a guard and removes none; `accept` is never refused by it.

`shedadapters.LatestRound(runDir)` returns the run directory's latest round, resolved exactly as `RecordCirclingDecision` resolves it (`ResolveRound` over `roundReviewPath`'s file name), 0 for none.
It lives in `circling.go`.
`TestRecordCirclingDecision_WritesPendingFile` asserts `LatestRound(dir)` equals each row's `wantRound` before recording.

`circlingDeps` gains `latestRound func(subdir string) (int, error)`, the latest round of the Bouncer run directory named by `subdir`.
It also gains `hardCap func(row string) (hardCap int, raise string, err error)`, the row's resolved hard cap and the way forward that raises it.
`circlingState.preRun` fills them with `shedadapters.LatestRound(filepath.Join(reviewsDir, subdir))` and `perchHardCap(target, row)`, both evaluated only when the verb calls them, so `accept` and the earlier refusals load no config.

`circlingVerb`, for `continue` only, after the Bouncer-row check and before `record`:

- It reads the round with `latestRound` and the cap with `hardCap`.
  Either error refuses as "%s; %s" with `circlingWayHardCap`: `way forward: fix the file or key the message names, then re-run the verb`.
- `hardCap > 0` and `round >= hardCap` refuses: "round N at <row> is at or past its perch's hard cap H, so no review round runs past it; way forward: run `lyx loom circling accept <slug>`, or raise the hard cap: <raise>, then run `lyx loom circling continue <slug>`".
  This reaches the refusal at a `hard-cap` escalation (whose round is always at or past the cap) and at a `circling` or `budget` escalation on such a round.
  A `continue` below the cap on a `circling` escalation records as before.
  The slug is the verb's `slug`.

`perchHardCap(location, row)` resolves under `fabricengine.ConfigDirs(location)`:

- For `loomshed.NameDarnBouncer` it loads darn.yaml through `darnengine.LoadConfig` and returns `ReviewHardMaxBounces` with `run "lyx config darn --set review_hard_max_bounces=<n>" from the prime`.
- For a row in `loomPerches` it loads loom.yaml through `loomengine.LoadConfig` and takes `cfg.Caps(perch)`.
  It returns `Hard` with `loomengine.CapRoute(dirs, HardKey)`, so a hard cap from the task's `run_settings` names the local-layer command and `lyx board upsert`.
- For any other row it returns an error naming the row, a wiring guard: `bouncerSubdir` has already confirmed a Bouncer row, and every Bouncer row is in one of the two sets.

`circlingWayNot` and `refusal-spec.md`'s "circling: latest round not escalated" row say "(a circling, hard-cap or budget escalation)" instead of "(a CIRCLING judgement or a spent review budget)".
`circling.go`'s header comment adds the hard-cap refusal to the verbs' preconditions and the three causes.
`loomCLI.circlingCmd`'s group, `accept` and `continue` Long texts name the three causes.
`continue` says it runs one more round only below the perch's hard cap and is refused at or past it, and drops "after a budget escalation that grants exactly one more round".
`internal/loomcli/doc.go` gains one sentence: `circling continue` refuses at or past the perch's hard cap with `circling accept` or raising the cap as its way forward.

`refusal-spec.md` gains, beside the circling rows:

- "circling: continue at the hard cap": `lyx loom circling continue` finds the latest round at or past its perch's hard cap, a correctness halt.
  Its way forward: `lyx loom circling accept <slug>`, or raise the hard cap with the command the message names for the layer its value comes from, then `lyx loom circling continue <slug>`.
- "circling: hard cap unresolved": `lyx loom circling continue` cannot read the run directory's latest round, or loom.yaml or darn.yaml fails to load, a correctness halt.
  Its way forward: fix the file or key the message names, then re-run the verb.

Tests:

- `circlingFake` gains `latest int`, `latestErr error`, `hardCap int`, `raise string` and `hardCapErr error`, wired by `circlingFake.deps`; a zero `hardCap` means no cap, so existing rows run as before.
- `TestCirclingVerb_Records` becomes a table: accept and continue as today, plus "continue below the hard cap on a circling escalation" (latest 3, hard cap 4, cause circling) recording with cause `circling`, and "accept at the hard cap" (latest 4, hard cap 4) recording.
- `TestCirclingVerb_Refusals` gains a `decision` field, defaulting to accept, and its prefix check uses that decision.
  New continue rows:
  - "continue at the hard cap": latest 4, hard cap 4, raise `run "lyx config loom --set review_hard_max_bounces=<n>" in the task worktree`; wants "at or past its perch's hard cap 4", `lyx loom circling accept task-a`, the raise text and `lyx loom circling continue task-a`.
  - "continue whose hard cap comes from run_settings": raise is the local-layer text with `--layer local` and `lyx board upsert`, which must appear verbatim.
  - "hard cap unresolved" (`hardCapErr`) and "latest round unreadable" (`latestErr`), each wanting "fix the file or key the message names".
  Every refusal row asserts nothing was recorded.
- New `TestPerchHardCap` resolves over `hubLocation` with configs seeded as the wiring tests seed them:
  - Plan-Bouncer with `plan_hard_max_bounces: 6` in the worktree's loom.yaml returns 6 and `lyx config loom --set plan_hard_max_bounces=<n>`;
  - Discussion-Bouncer with a local `review_hard_max_bounces: 5` written at `configengine.LocalFile(anchor, "loom")` returns 5 and the `--layer local` route with `lyx board upsert`;
  - Darn-Bouncer over `seedDarnHubConfig` returns the template's 4 and `lyx config darn --set review_hard_max_bounces=<n>`.
  `cmd/lyx/testdata/test-budget.yaml`'s `internal/loomcli` row rises from 221 to 222 for it.

**ImpactSummary:** The circling verb group gains one continue-only precondition and two deps; accept, the other refusals and the integration pre-run test are unchanged, and the shedadapters export is a read-only round lookup.

**Commit:** 8: circling continue refuses at the perch's hard cap
