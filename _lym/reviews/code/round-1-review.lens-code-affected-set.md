```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** For each change since the settle, I list what reads or asserts it and whether the diff covers it.

- **F1, lazy `programName` in `internal/loomcli/decision.go`.**
  - The set is the `decisionDeps.programName` field (`decision.go:226`), `noticeFits` (`decision.go:303`) and its two callers (`decision.go:344`, `decision.go:667`), `preRun` (`decision.go:787`) and the test fake (`decision_test.go:140`).
  - Covered. Only these uses of the field exist, and all are updated. No doc or PATTERN text names the field.
- **F2, probe revert.** The net effect of `b797614f` plus `4d56042d` is nil; `deliver.go` is back at the settled text. Covered.
- **F3, `tools/sandbox/SANDBOX-CREEL-SUITE.md`.**
  - The grep for unquoted `from <...>` sender lines across `tools/`, `docs/`, `contracts/` and `internal/` found none left. The only hits were unrelated `--from` flags.
  - Covered.
- **F4, `internal/creelcli/cli_test.go`.** The no-daemon `hook-wait` row now advances the client clock through `AutoAdvance`, the same pattern as the existing row at line 434. No docs or fixtures are affected. Covered.
- **F5, `internal/shuttleengine/claudeengine/command_test.go`.** The assertion that an open session carries no `--append-system-prompt-file` matches D9. `claudeengine/doc.go` states the same. Covered.
- **Late `9ea69053`, `internal/fabriccli` tests.** `traceSinkMu` is declared in `testmain_test.go` and used by both test files in `package fabriccli_test`. Test-only, with no other set. Covered.
- **Parent merge and alignment-report claims (`internal/reedengine`).**
  - `composePaneLaunchLine` now chains the prelude, the build-server exports, the name exports, then `launchCmd`.
  - `launchCmd` is the shuttle launch command, which carries D9's `--append-system-prompt-file` and the hook-mark settings. The merge changed neither the call site nor the signature.
  - `reedengine/doc.go` holds both `proc.BuildServerOffEnv` (line 178) and the `CreelHooks` paragraph (line 230).
  - It also holds the parent's spawn-refusal line (line 291).
  - `Strand.CreelHooks` is unaffected by the `proc.NoteSpawnAgain` change in `overlay.go`.
  - Covered.
- **Parent merge, `internal/loomcli` darn wiring.** The `latestDarnOutcome` change (`darnwiring.go`) and the `loomAfterInterrupt` change (`looparm.go`) do not touch `decision.go`'s notice cap or `LineFits`. The report's "no conflict" claim holds. Covered.
- **PATTERN.md and `pattern/`.** `git diff 6b737198 8be3019c` over `PATTERN.md` and `pattern/` returned nothing, so the merge changed no PATTERN text. The report's claim about them holds. I read only the `git diff --stat` for `pattern/PATTERN-daemon-wakeups.md`, which is task-side and was already settled.
- **`shuttleengine.ErrAgentDied`, `shedtransient` and `shedverbs`.** These touch no creel code in `creelengine`, `creelcli` or `claudeengine`. I found no changed signature that composes with the creel call sites.

**Findings:** none.

**Tool calls (2):**
1. Grep for `.programName` and `programName:` across `internal/`, to find the users of the changed field. My context had only the diff hunks.
2. Grep of the docs trees for unquoted `[creel] #<id> from <...>` lines, to check that F3's sandbox fix left no stale form. My context did not show the other doc files.
