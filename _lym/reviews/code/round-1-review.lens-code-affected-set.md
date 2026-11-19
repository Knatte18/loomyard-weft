```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change in the diff I traced what it forces outside the lines it touches.

**Per-change affected set**

1. **`loopExitEvidence` signature, now `(LoopSpec, loopID, startedAt)`.** Covered.
   - Callers are `deadLoopStop` (`loopwait.go:596`) and `loopExitedEarlyStop` (`loopwait.go:634`).
   - Tests are the two integration call sites (`loop_unix_integration_test.go:211`, `:301`).
   - All are updated, and the doc comment is updated.
2. **New `LoopPIDRecord.StartedAt` field (`started_at`, `omitzero`).** Mostly covered, one gap.
   - Covered:
     - The writer in `runLoop` and the in-flight rewrite in `stepUntilStop` (`loop.go:414`, which copies `*pid` and keeps the field).
     - The `pairteardown` read-then-restore of `prior` (`internal/pairteardown/loop.go:80-92`, which round-trips the field).
     - The `landingdeps.go:787` reader, which is unaffected by an extra field.
     - The teardown mark, which has no start, so the zero value fits.
     - `doc.go:276`, which mentions the recorded start.
   - Not covered: the package doc's list of what the pid file holds (`doc.go:155-157`). See F1.
   - No spec under `contracts/specs` describes the pid file.
3. **Unmarked-exit wording, now cause-specific through `unmarkedExitCause`.** Covered.
   - The `loopwait_test.go` rows (`:649`, `:658`, `:668`, `:678`) are updated or added.
   - The package doc is at `doc.go:276-280`.
   - No stencil or spec quotes the old wording.
4. **Early-exit trace messages replaced by `endWithoutEnvelope`, with a new loop-log mark.**
   - Messages removed: "loop refuses to step", "could not take the loop lock", "could not create its directory", "could not write its pid file".
   - No test, doc or stencil asserts the removed messages.
   - `docs/logging-inventory.md` lists none of them, and it is a measurement report anyway.
   - Marks from another loop id are excluded from a loop's output by `loopLogLinesOf` (`loopwait.go:278`), so a lock-loser's mark cannot pollute the holder's excerpt.
   - The new package-doc wording covers only part of the new behaviour. See F2.
5. **Envelope-write failure now marks the loop log.** Covered by the `runLoop` doc and `doc.go:226`, and asserted by the new test's second row.
6. **New `LoopSpec.BootTime` seam.** Covered by the field doc, `doc.go:277` and the `loopwait_test.go:707` seam use.
7. **New `proc.BootTime` on Linux and Windows.** Covered: `internal/proc/doc.go` is updated, and `unmarkedExitCause` is the only caller.
8. **The extra `Loop.Now` read at loop start.** Covered.
   - Tests that panic on the clock are updated: `loop_test.go:548` and `loop_unix_integration_test.go:140`.
   - The fake-clock tests (`loop_test.go:1415`, `:1688`) take one extra read and are not reported failing by the gate.
9. **`loopAgreesWithState` now treats held beside busy/background as agreement.** Covered.
   - Updated: the `shuttleengine/doc.go:1026-1032` mapping text, the function doc, and the `waitshadow_test.go` row.
   - No other doc or spec describes the mapping.
   - `docs/research/.../calls.tsv` is a research artifact.
10. **Repository rules.**
    - No PATTERN changes are declared, so none are owed.
    - No commit in the range carries a `Pattern-Departure:` or `Loosened-Check:` trailer.
    - Two test seams were adapted (the clock lets its first read through), which is not a loosened check.
    - PATTERN-test-speed: the new loop tests omit `t.Parallel`, which the file-level comment at `loop_test.go:3` covers. `TestBootTime` is parallel.

**Findings**

- **F1**
  - Severity: LOW. Class: consistency.
  - Location: `internal/shedverbs/doc.go:155-157`.
  - Summary: the package doc lists what the pid file holds as "the loop id and the loop's own pid and start time", plus the in-flight child's record. That list omits the new wall-clock `started_at`, and "start time" there means the process start-time token. The list now under-describes the file shape, and a reader cannot tell the two starts apart.
  - Evidence: `doc.go:155-157` against `looppid.go:28-30` (the new `StartedAt` field).
- **F2**
  - Severity: NIT. Class: consistency.
  - Location: `internal/shedverbs/doc.go:224-226`.
  - Summary: the new package doc says a loop that ends before its first step marks the reason, and that "such an end comes from a refusal or a lock another loop holds". `endWithoutEnvelope` also marks a directory it cannot create and a pid file it cannot write (`loop.go:213`, `loop.go:244`). The doc's list of mark sources is incomplete.
  - Evidence: `doc.go:224-225` against `loop.go:213` and `:244`.

**Tool calls**

1. Grep across Go, md and tmpl files for the changed messages, wordings and `BootTime`: to find docs, tests and stencils outside the manifest that read them.
2. Grep of `shedverbs/doc.go` and `contracts/specs` for the pid file's described shape, and of `contracts`, `docs` and `pattern` for the shadow mapping, plus `doc.go:205-225`: the manifest had not read the package doc's pid-file and early-exit text.
3. Read `doc.go:150-160` and grepped `docs/logging-inventory.md` for the removed log messages: to confirm the pid-file description and whether any inventory names the removed messages.
