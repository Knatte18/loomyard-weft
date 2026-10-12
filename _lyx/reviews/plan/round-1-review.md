---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/16-batten-cleanup.md:58
    summary: Card 16 decides whether a run directory is tracked by probing only status.json and seed.json at the records HEAD, so a run directory whose seed is committed but whose status is not falls into no branch of step 3, and D9 requires that such a tracked slug is skipped, reported and never emptied by the orphan rule.
    fix: In step 3 of card 16's Intent, add a branch after the absent-but-tracked one saying that when the run directory is absent on disk and Committed finds shedrun.SeedRel but not shedrun.StatusRel, the slug is skipped and reported with the reason tracked run directory with no committed status, its deletion is left uncommitted and its scratch is never emptied. Add a row for that case to cleanup_test.go's table in the Tests paragraph.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/16-batten-cleanup.md:43
    summary: The prime-only refusal that battencli.Cleanup applies is reached by no named test. cleanup_test.go drives the unexported cleanup, cleanup_integration_test.go runs only from the prime, and card 20 calls RefuseUnlessPrime itself before the batten segment, while resolvePersistentPreRun skips the arm for the new verb.
    fix: In card 16's Tests paragraph, extend cleanup_integration_test.go so the hubforge hub also has a task pair, and add a step that runs RunCLIIn(<task pair worktree>, out, "cleanup", "--apply") and asserts an error envelope naming the prime worktree, with the done run directory still present on disk and at the records HEAD.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/06-testtiming-run-directory.md:7
    summary: Card 6's Intent removes the per-package temp directory that measurePackageResources takes today as its tmpDir parameter and has the function compute the marker from importPath itself, but the Edit bullet carries no re-sign arrow. leftoverProcesses also changes its parameter from a directory to a marker.
    fix: In card 6's Edit list, replace `cmd/testtiming#measurePackageResources` with `cmd/testtiming#measurePackageResources` -> `func measurePackageResources(tags, importPath string, env []string, parallel int) (resourceRow, error)`, and replace `cmd/testtiming#leftoverProcesses` with `cmd/testtiming#leftoverProcesses` -> `func leftoverProcesses(marker string) []string`.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/02-tmuxkit-run-directory.md:50
    summary: Card 2's ImpactSummary says afterRun has two callers, but it has three. The third is the afterRun call in TestCheckSockets' subtest at internal/testkit/tmuxkit/tmuxkit_test.go:294, which the Intent mentions only for its MkdirTemp change, not for the new remove argument.
    fix: In card 2, change the ImpactSummary to say afterRun has three callers, all on this card (Main, TestAfterRun_LeftoverProcesses and TestCheckSockets' afterRun subtest). Add a sentence to the Intent saying TestCheckSockets' afterRun subtest passes a remove that deletes its directory.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/20-cleanup-verb.md:50
    summary: Card 20's board segment rebuilds the hub board by hand (LoadConfig over fabricengine.BoardDir, Path set to the board directory, then ApplySkipEnv), which is exactly what the existing boardengine.OpenHub(hubPath) already does.
    fix: In card 20's segment 2, replace the hand-built config with boardengine.OpenHub(location.HubPath) followed by Board.Prune(apply), and use the same board for the scratch segment's ListTasksFull. In Uses, replace internal/boardengine#LoadConfig and internal/fabricengine#BoardDir with internal/boardengine#OpenHub.
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/20-cleanup-verb.md:64
    summary: No test reaches the acceptance criterion that the log segment sweeps with logger.yaml's bounds. Card 13 pins LoadOrDefault against Load, but card 20's integration test seeds no trace files and no logger.yaml, so its per-anchor wiring of LogsDirAt and LoadOrDefault for both the prime and _board is never exercised.
    fix: In card 20's cleanup_integration_test.go description, also seed the prime's logger.yaml with a short max-age bound, plus one trace group in the prime's logs directory that is older than that bound and younger than the default. Assert that the dry run lists that group under the logs segment and --apply removes it.
---

### [MEDIUM] Tracked run directory with a committed seed but no committed status has no disposition

**Location:** `_lyx/plan/16-batten-cleanup.md:58`

**Issue:** D9 says a run directory that is deleted on disk but still tracked is a candidate only when its committed `status.json` decodes to `done`, and is otherwise skipped and reported, and that such a slug's scratch is never emptied by the orphan rule.
Card 16 approximates "tracked" by two `Committed` probes.
The tracked branch fires only when `StatusRel` is found, and the orphan branch only when neither `StatusRel` nor `SeedRel` is found.
A run directory whose `seed.json` is committed and whose `status.json` is not matches neither branch, so the implementer is left to guess.
Treating it as an orphan would empty the scratch of a tracked slug, which D9 forbids.

### [MEDIUM] battencli.Cleanup's prime refusal has no test that reaches it

**Location:** `_lyx/plan/16-batten-cleanup.md:43`

**Issue:** `Cleanup` is the guard for `lyx batten cleanup`, because the card makes `resolvePersistentPreRun` return early for the verb, so `arm`'s non-prime refusal never runs for it.
The card's tests never run the verb outside the prime.
Card 20's integration test does run `lyx cleanup` in a task pair, but card 20 applies `RefuseUnlessPrime` before any segment, so that run never reaches `battencli.Cleanup`.
The rubric's reachability rule asks for a named test that reaches every prescribed guard.

### [MEDIUM] measurePackageResources and leftoverProcesses change signature without the re-sign arrow

**Location:** `_lyx/plan/06-testtiming-run-directory.md:7`

**Issue:** Today the signature is `measurePackageResources(tags, importPath, tmpDir string, env []string, parallel int)` (`cmd/testtiming/resources.go:155`), and `runResources` passes it the `ttr-*` directory.
Card 6 deletes that directory and has `measurePackageResources` build the marker from the import path and the pid, so `tmpDir` goes away.
`leftoverProcesses(dir string)` (`resources.go:405`) becomes `leftoverProcesses(marker)`.
Neither bullet carries the arrow the plan template requires for an Intent-changed parameter list.

### [LOW] Card 2 miscounts afterRun's callers

**Location:** `_lyx/plan/02-tmuxkit-run-directory.md:50`

**Issue:** `afterRun` is called at `tmuxkit.go:89` (`Main`), at `tmuxkit_integration_test.go:270` (`TestAfterRun_LeftoverProcesses`) and at `tmuxkit_test.go:294` (the "afterRun reads the sockets before the sweep removes them" subtest of `TestCheckSockets`).
The card does target `TestCheckSockets`, but its Intent mentions that test only for its `os.MkdirTemp` site, and the ImpactSummary claims two callers.
A fixer working from the Intent would miss the third call site, and the package would not compile.

### [LOW] Board segment duplicates boardengine.OpenHub

**Location:** `_lyx/plan/20-cleanup-verb.md:50`

**Issue:** `boardengine.OpenHub` (`internal/boardengine/config.go:176`) already loads the config from `fabricengine.BoardDir(hubPath)`, sets `Path` to the board directory and applies `ApplySkipEnv`.
Card 20 spells the same steps out inline, so the hub-board opening now exists in two places.

### [LOW] The log segment's bound choice is never exercised end to end

**Location:** `_lyx/plan/20-cleanup-verb.md:64`

**Issue:** The decision record's acceptance criteria include "The log segment sweeps with `logger.yaml`'s bounds".
Card 13 tests `LoadOrDefault` and `SweepDir` on their own.
Card 20 is where they are wired per anchor, for the prime and `_board`, but neither its `cli_test.go` nor its integration test seeds a log or a `logger.yaml`.
A mistake in that wiring, such as passing the wrong anchor to `LoadOrDefault`, would therefore pass every planned test.

## Rejected

- Card 16's dry run creating lock files: the step 1 sentence "In a dry run no lock file is ever created: a lock whose file is absent counts as free and is not taken" is general, so it also covers the step 2 status lock. Not a defect.
- Card 6's environment marker as unlicensed scope: no. D5 moves testtiming's directories into the run directory, but handing a child the `ttr-*` directory as `TMPDIR` would nest a second root and run directory under it, and its socket path, about 133 bytes, would exceed `checkSocketPath`'s 107-byte limit (Constraint 1). Dropping the directory and attributing leftovers by a marker is the consequence the decision record forces.
- Card 16's orphan-scratch rule and `push_error`: D9 licenses both. The orphan rule is D9's own bullet, and `push_error` is how "a push failure is reported" reaches the envelope.
- Card 5's `lyxtemp.Remove()` in `cmd/lyx`'s `main`: D1 licenses it ("The owner removes its run directory explicitly, unlocking first, when it ends normally").
- Card order: every `Uses` points at an existing symbol or at an earlier card (1 to 2–6 and 20, 14 and 15 to 16, 19 and 20, 16 to 17, 13, 16, 18 and 19 to 20). Card 1's and card 3's use of `tmuxkit.Main` uses the existing function. Not a defect.
- TestMain coverage in cards 7–9: a scan of every test package directory shows that the 79 packages with no `TestMain` match the three cards' lists exactly. `gitoracle` is the only `TestMain` that does not call the kit, and card 9 covers it. The external package clause for `agentname`, `lock` and `proc` matches `tmuxkit`'s transitive imports (`agentname`, `proc`, and `lock` through `lyxtemp`).
- Bare-temp sites: all 21 `os.MkdirTemp`/`os.CreateTemp` calls with `""` in the tree are on cards 2–6, so card 12's empty allowlist holds. `shuttlecli`'s `makeKeepDir` passes `os.Getenv(...)`, which D6 allows.
- Granularity: card 16 is one surface, the batten cleanup verb with its seam, helper and tests. Splitting cards 7–9 by letter range keeps mechanical batches reviewable, and card 11's rename of the guard test is reasonable as its own card. Not a defect.
- Verify coverage: the tree is one module (`tools/` and `contracts/` included), so `./...` covers every targeted package, and the section follows the template's order.
- `git add -- <removed tracked dir>`, which `CommitAnchoredPaths` reaches through `StageAndCommit`, stages the directory's deletion. A scratch repository confirmed it, so card 16's commit of `RunDirRel` works.
