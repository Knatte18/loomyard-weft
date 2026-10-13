---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/21-batten-wiring.md:3
    summary: Card 21 makes the batten-watched marker three lines, but the integration test that pins its exact one-line content is not among the card's targets, so go test -tags integration fails on this card and at the plan verify.
    fix: In card 21's Edit list, add `internal/battencli#stepMarkWatched_HoldsTheMarkerOnlyWhileItsNoticesReachTheDriversParent` (internal/battencli/lifecycle_integration_test.go:1265-1326). In the Tests list, add a bullet saying its held rows now expect `shedrun.BattenWatchedContent(os.Getpid(), <BattenDir of the slug>, battenrecipe.NameRunShed)` in place of the pid line it compares against today (`strconv.Itoa(os.Getpid()) + "\n"`). The combined validate-plan --overlay run with this target admits it.
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/02-loom-config-keys.md:45
    summary: Card 2 adds a LoadConfig refusal for a tackler value that is not a model-spec, but gives it no refusal-spec row, which PATTERN-refusal-way-forward requires; refusal-spec.md has the matching arbiter row, config arbiter not a model-spec.
    fix: In card 2's Intent, extend the refusal-spec.md sentence to add a third loom-section row beside the arbiter model-spec row, titled config, tackler not a model-spec, worded as the arbiter row is. Trigger is `loom.yaml`'s or a task's run_settings `tackler` being non-empty and not a model-spec, or one the registry does not define. Class is correctness halt. Way forward is to set `tackler` to a model-spec the registry defines, or to empty to take `haiku`. `contracts/specs/refusal-spec.md` is already an Edit target, so no target changes.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/14-drive-handover.md:23
    summary: D5 says a dirty tree has an uncommitted or untracked path outside the run's records, but the clean-tree check in cards 14, 15, 20 and 21 reads fabricengine's DirtyPaths, which lists tracked paths only, so a new untracked file never blocks a hand-over or batten's binary re-step.
    fix: Make the hand-over and batten re-step clean-tree read see untracked paths. In card 15's worktree bullet, build `DirtyPaths` for the hand-over from `fabricengine.Open(location)`'s `Status()`, which lists tracked-modified, staged and untracked paths per side. Join each entry to its side's checkout root to get an absolute path, then filter through `loomengine.PathsOutsideRunRecords`. Make the same change in card 21's `ChildDirtyPaths` bullet. In cards 14 and 20, change the seam text from tracked uncommitted paths to uncommitted or untracked paths, and in card 20 add an untracked-file row to the spawns-nothing list. Card 7's tamper snapshot stays on the tracked read, because D7 names the existing dirty-paths read for it. Text-only change, no target edits.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/12-drive-loop.md:58
    summary: A moved run that card 12 decides afresh through stopFromStatus has no saved envelope, yet card 13 passes the saved envelope to Arbiter and Tackler; newArbiterCall refuses an unreadable envelope, so a moved run at an awaiting escalation or a tackler stop always hands back instead of reaching the agent D12 asks for.
    fix: In card 12's recheck bullet, say that a moved status decided through `stopFromStatus` is saved through `SaveEnvelope` as a synthesized envelope. That envelope carries `state`, `reason` empty, and a `loop` object with `current_producer` and `history_length` from the status, and its path becomes the stop's `EnvelopePath`. Add a `TestGoDriver_Settle` row to card 13 under Moves and tampering where a re-read status moved to an awaiting stop with a pending escalation calls `Arbiter` with a readable envelope path. Text-only change, no target edits.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/15-drive-verb.md:40
    summary: Card 15's Bound for the new verb says drive does nothing a lyx loom step caller cannot, but drive spawns tacklers, which have no verb (A2), as well as arbiters and successor drivers, and performs the verify goto, so the stated bound is false and would carry into doc.go.
    fix: Replace card 15's Bound sentence with the real bounds. drive steps only through `shedverbs.RunUntilStop` and `RunGoto`, and its goto is only the matching verify record's target, once per run. Its spawns are capped by `tackler_max_spawns` and `arbiter_max_spawns` per run, plus one successor per stop key and build, over a clean tree only. It moves nothing while a pause covers the slug, and the driver lock caps it at one driver per run. Text-only change.
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/03-run-record-paths.md:11
    summary: Card 3 creates and tests loomengine.IsTacklerStrand, but no card calls it; batten's tackler hold in card 21 matches the marker's strand GUID, not names, so the function is dead code that no Decision licenses.
    fix: Remove the `plan:internal/loomengine#IsTacklerStrand` Create bullet from card 3. Drop `IsTacklerStrand(name string) bool` from the tackler.go bullet list, and drop the `IsTacklerStrand` rows from `TestTacklerRecords`. The combined validate-plan --overlay run with this bullet removed reports no new finding.
---

### [BLOCKING] Card 21 misses the integration test that pins the watched marker's content

**Location:** `_lyx/plan/21-batten-wiring.md:3`

**Issue:** Card 21 switches `markBattenWatched` to write `shedrun.BattenWatchedContent(os.Getpid(), scratchDir, producer)`, which holds three lines.
`internal/battencli/lifecycle_integration_test.go`'s `stepMarkWatched_HoldsTheMarkerOnlyWhileItsNoticesReachTheDriversParent` compares the marker byte for byte against `strconv.Itoa(os.Getpid()) + "\n"` on its held rows (lines 1316-1319).
The card's targets do not list it, so the card's own integration tier fails, and so does the plan's `go test -tags integration ./...`.
The plan template's fourth card rule requires every test asserting a reworded output to join the card that changes it.

### [BLOCKING] The tackler model-spec refusal has no refusal-spec row

**Location:** `_lyx/plan/02-loom-config-keys.md:45`

**Issue:** Card 2's `LoadConfig` refuses "a `tackler` that is non-empty and does not parse as a model-spec", and `TestLoadConfig_Refuses` reaches it.
The refusal-spec paragraph adds only `config: driver_kind not llm or go` and `config: tackler knob negative`.
`contracts/specs/refusal-spec.md:246` holds `config: arbiter not a model-spec` for the matching arbiter refusal, so the tackler refusal is a new loom refusal landing with no spec row, which violates `PATTERN-refusal-way-forward` and the decision record's constraint on it.

### [MEDIUM] The clean-tree check drops D5's untracked paths

**Location:** `_lyx/plan/14-drive-handover.md:23`

**Issue:** D5 says "the task worktree has no uncommitted or untracked path outside the run's own records directories".
`fabricengine.Fabric.DirtyPaths` (`internal/fabricengine/diff.go:99-111`) runs `git status --porcelain -z --untracked-files=no`, and its doc says untracked files are not listed.
Cards 14 and 20 define the seam as "tracked uncommitted paths", and cards 15 and 21 wire it to that read, so the plan silently drops the untracked half of D5 and never states the gap as a bound.
A hand-over or batten re-step can then run over a worktree that holds a new untracked file.
`Fabric.Status` (`gitrepo.Repo.WorktreeChangedFiles`, which covers untracked files) is an existing read that covers it.

### [MEDIUM] A status-derived stop carries no envelope into the agent paths

**Location:** `_lyx/plan/12-drive-loop.md:58`

**Issue:** Card 10 says `stopFromStatus` builds a stop "from a status alone, with `FromStatus` true, no kind and no reason", and card 12 decides any moved status that is not a goto afresh through it.
Neither card gives that stop an `EnvelopePath`.
Card 13's `arbitrate` calls `Arbiter` "with the saved envelope", and `newArbiterCall` (`internal/loomcli/arbiter.go:160-178`) refuses when it cannot read `in.EnvelopePath`.
So a moved run at an awaiting stop with a pending escalation is always handed back through a refusal, not arbitrated.
The tackler gets `none` for its envelope.
D12's "acts on the fresh state as a new stop" is therefore only partly carried, and no test row reaches the case.

### [LOW] Card 15's bound for the new verb is wrong

**Location:** `_lyx/plan/15-drive-verb.md:40`

**Issue:** "the verb does nothing a `lyx loom step` caller cannot, since it only steps and settles through the same verbs" is false.
`drive` spawns the tackler, which A2 says has no CLI verb, along with the arbiter and a successor driver, and it performs a goto.
The real bounds are spread across cards 7, 13 and 14, but the verb card's attack-surface statement, which card 15 also feeds into `doc.go`, names the wrong one.

### [LOW] `IsTacklerStrand` has no consumer

**Location:** `_lyx/plan/03-run-record-paths.md:11`

**Issue:** No card after card 3 uses `loomengine.IsTacklerStrand`.
The arbiter's counterpart has consumers in `internal/loomcli/decision.go:200` and `internal/battencli/wire.go:138`, but no card plans a tackler equivalent of either, and card 21 tells a live tackler apart by the marker's strand GUID.
The symbol and its test rows are scope no Decision licenses.

## Rejected

- Verify section: it covers every targeted package through `./...`, runs the integration tier, compiles the `tmux` and `llm` tiers only, and follows the template's order.
- Card order across cards 10 to 15: every handle a card uses is created by an earlier card. The forward mentions (card 12 naming cards 13 and 14, card 14 naming card 15's park-marker removal) are prose, not dependencies.
- Re-sign arrows: every Edit whose Intent changes a signature carries the arrow (`mustSpawnDriver`, `resolveSeedDriver`, `childDriverOf`, `markBattenWatched`). The other edited members keep their signatures: `goDriverDeps` is a struct, and `goDriver.run`, `goDriver.facts`, `runDriverSpawnAndWait`, `waitRunLockFree` and `findDriverPIDs` are unchanged.
- Reachability: each refusal or hand-back that cards 7, 8, 11, 12 and 13 prescribe has a named input and test row. These include the spent and zero budgets, `errTacklerResumeReturned`, the applied-mark escalation, `tampered`, busy, moved, a consume refused as moved, and a drive lock that is already held. No upstream check rejects those inputs first.
- CLI shape: `drive` is hidden with the internal audience, `cobra.NoArgs`, and a lower-case `Short` with no final period, which fits `PATTERN-cli-cobra`. The empty `--child-driver` default in card 18 changes no command shape.
- Test budget: battenshed counts 109 top-level tests against a row of 110 (TestMain is excluded), so card 9's claim of room for one test holds.
- Tackler `Interactive: false` matches `arbiterSpec`, which is also a tmux strand in the pair's session, so it does not contradict D7's interactive strand.
- The probe race between start's or batten's driver-lock probe and `drive`'s non-blocking acquire copies the run lock's existing `TryAcquireWriteLock` probe pattern, so I am not raising it.
- The tackler tamper snapshot reading tracked paths only follows D7, which names the existing dirty-paths read. Card 7 states that as its bound.
