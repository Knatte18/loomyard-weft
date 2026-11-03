```yaml
lens: code-affected-set
outcome: reported
```

**Per-change set (code-affected-set lens)**

1. `commit-records` envelope gains `push_stopped`.
   - Set found: `internal/loomcli/commitrecords.go:29-52`, `internal/loomcli/doc.go:278` and the new `doc.go:288` paragraph, and `internal/loomcli/wiring_commitstatus_integration_test.go:283-400`. Searching the repo for `push_skipped` found no stencil, spec or other doc that lists the envelope's keys.
   - Docs: covered. The function comment and `doc.go` both name `push_stopped`.
   - Tests: not covered. No test asserts the `push_stopped` envelope branch.
2. `Spawn`, `Push` and `PushTask` now call `check.guard()`.
   - Set found: `wiring.go:213-230`, `landingdeps.go:338-344`, and the callers `landingshed/prreview.go:201,553` and `finalize_pr.go:215`.
   - Behavior: Finalize skips the check, so `finalize_pr.go:215` is unaffected. `PushTask` as a seam is listed in `wiring_seams_test.go:40` and `battencli/wire_seams_test.go:99`. Neither lists a new seam, so both stay valid.
   - Other push paths: no unguarded loom-run push exists in `loomcli`, `loomengine`, `shedrecipe`, `landingshed` or `statuscommit`. `pawlwiring.go:94` (`SpawnPairPush` at beam-up) and the batten paths (`battencli/commitstatus.go`, `cancel.go`, `cleanup.go`, `readopt.go`, `tenderengine/resume.go:221`) are not a loom run's push.
   - The existing integration test now runs the guard through the real `loomCommitStatusDeps` (`wiring_commitstatus_integration_test.go:47,283,311,349,400`). Whether the real reconcile or the skip conditions change what those tests assert is unverified.
3. `ownerPauseGrace` renamed to `pauseGrace`, plus the new `PausePair` in `tendercli/deps.go`.
   - Set found: the only users are `deps.go:211,220,234`. Other `pauseGrace` identifiers in `hibernateengine` and `orchengine` tests are separate packages.
   - Docs: `tendercli/doc.go:35` is updated. Covered.
   - Tests: `PausePair` has no test. `cli_test.go:210` covers only `PauseOwnedElsewhere`.
4. `PATTERN.md` and `pattern/PATTERN-owner-machine.md`.
   - Set found: the `PATTERN.md` entry, the background header, the new bullets and the Enforcement line. `battenshed/doc.go:166` is updated.
   - Declared changes: the directive declares none.
   - Departures: the commit has no `Pattern-Departure:` trailer.
   - Wording: partially covered, see finding 1.
   - The `PATTERN.md` entry has no `(test)` marker, as before.

**Findings**

1. LOW, consistency, `pattern/PATTERN-owner-machine.md:3,15-16,22`.
   - The new push rule is in the title line and the bullet list, but the verdict sentence still reads "gates a resume, a wake or a landing".
   - The "Origin reconcile first" section and the "Passes" section do not mention a push.
   - The "## Origin reconcile first" section is where the reconcile ordering lives.
   - Evidence: diff hunk `pattern/PATTERN-owner-machine.md` lines 3 and 15-16 untouched.

2. LOW, consistency, `pattern/PATTERN-owner-machine.md:38`.
   - Enforcement now says the loom tests "reach" "loom's pre-push stops".
   - `prepush_test.go` drives only `guard()` over stubs.
   - No test reaches the three wired guards or the `push_stopped` envelope, so the claim overstates the coverage.
   - Evidence: `prepush_test.go:315-452` and the absence of any other reference (grep for `push_stopped` and `errPushStopped` in tests).

No finding on stencils: the diff touches none. No finding on the other push paths: they are not a loom run's push.

**Tool calls**
1. Grep for `push_skipped` and push-spawning symbols across loom packages, to find envelope docs and unguarded push paths. My context held no such search.
2. Grep tests for `commitRecordsVerb`, `push_skipped` and `ownerPauseGrace`, and the pattern docs for owner-machine mentions. I needed the tests and the other pattern mentions.
3. Grep of the existing commit-status integration test, to see whether it reaches the new guard.
