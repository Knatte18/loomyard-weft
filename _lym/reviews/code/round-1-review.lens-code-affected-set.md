```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change I list what reads, calls or asserts it and whether the diff left it wrong.

**Per change: set found, covered or not**

1. **Drift step `Dir` default removed and its base moved to the worktree root.**
   - `seats.go:136` is NOT covered. `captureReading` still renders ``run in `<Dir>` under the anchored directory``. The baseline agent's prompt therefore states the old base. With an unset `dir` it renders an empty ``` `` ```. No test asserts that wording, so nothing caught it.
   - `internal/driftcli/doc.go:14` is NOT covered. It still says the engine is told the anchored directory "where … a command step's relative dir is joined on".
   - The `CommandStep.Dir` doc, `driftengine/doc.go`, the template text, `wiring.go` and the `CaptureRequest`/`CompareRequest`/`stepRun` field docs are covered. `steprun.go:165` uses `AnchorDir` only as the slot holder's worktree, which is fine.
   - Existing config tests are covered: the diff updates them to expect an empty `Dir`.
2. **`WorktreeDir` added to the capture, compare and step requests.** Covered. All five `RunCapture`, `RunSideCapture` and `RunCompare` construction sites fill it: `driftcli` at `cli.go` 241, 304 and 424, and `loomcli/driftwiring.go` at 151 and 194. `driftwiring.go:217` and `driftcli/cli.go:352` build the capture-child spawn request, which only needs `AnchorDir`.
3. **`PairDir` seam moved from `AnchorPath()` to `WorktreePath()`** (`wire.go:1111`, `dobbynotice.go:107`). One finding, F-A below.
4. **`configreg.Module.CheckDirs` and `checkHubModule`.**
   - The registry, the `setHubWide` path and the `copyStagedToHub` path are covered.
   - `configcli/doc.go` is covered.
   - The non-hub-wide `setChecked` and `editMachine` paths still call `CheckLoad` only. That fits, since both modules are hub-wide and `CheckDirs` is only set on those.
5. **Templates and `gateslot/doc.go`.** Covered. Both templates and `gateslot/doc.go` state the base and the refusal.
6. **PATTERN and `pattern/` entries.**
   - The diff declares no PATTERN change.
   - I found no entry or background file for command-dir resolution.
   - `PATTERN-refusal-way-forward` binds `internal/webster*`, `shed*` and `loom*`; the new refusal lives in `driftengine` and `gateslot`, so it is not bound.
   - `PATTERN-config-layers` is untouched: `CheckDirs` calls the strict `LoadConfig` inside `gateslot` and `driftengine`.
   - Nothing to flag.
7. **Commit messages.** There is no `Pattern-Departure:` trailer and none is needed.

**Findings**

- **F-A, MEDIUM, consistency.** `internal/battencli/lifecycle_integration_test.go:1568-1584`. `stepPairDirNamesTheTaskWorktree` still says "the task worktree's own anchor" and asserts `PairDir()` equals `taskLocation.AnchorPath()`. The diff changed the seam to `WorktreePath()` (`wire.go:1111`) and did not touch this test.
  - It passes only while the fixture hub is anchored at the root, where the two paths are equal.
  - It does not pin the change: it can't fail if the seam reverts to the anchor.
  - Fix: assert `taskLocation.WorktreePath()` and rename the step and its comment. Ideally use a subdirectory-anchored fixture so the two paths differ.
  - The `PairDir` doc in `battenshed/deps.go:216` ("task worktree directory") and the `PairDir` doc in `dobbyengine/runnotice.go:72` already match the new behavior.
- **F-B, MEDIUM, scope.** `internal/driftengine/seats.go:136`. The Drift-Baseline agent's prompt says the capture runs "in `<dir>` under the anchored directory". That is the old base, and it renders an empty name for an unset `dir`.
  - Fix: render the resolved directory. For example, take the worktree root and anchor into `BaselineInputs` or `captureReading` and print `gateslot.ResolveCommandDir(...)` relative to the worktree root. Otherwise say "relative to the worktree root, or the anchored directory when no dir".
  - Add a case to the `BaselineTable` test for an empty `Dir`.
- **F-C, LOW, consistency.** `internal/driftcli/doc.go:14`. The text still says the anchored directory is where "a command step's relative dir is joined on".
  - Fix: say the engine is told the worktree root, which a step's relative `dir` is joined onto, and the anchored directory, where an unset `dir` runs and the capture child starts.

**Tool calls**
1. `grep -rn "PairDir" internal --include=*.go`: to find the producers, consumers and tests of the `PairDir` seam. My context held only the diff lines for it.
