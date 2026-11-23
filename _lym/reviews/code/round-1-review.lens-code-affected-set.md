```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens on the test-only diff in `internal/loomcli/gotohook_integration_test.go`.

This lens finds nothing to file.

**What each change touches, and whether it is covered**

1. **The `TestGotoSpec_AdmittedHookClearsThePRReviewRecordOnlyForThatRow` table gains a `held` and a `wantErr` field, and a new row.**
   - Nothing outside the test reads the table. The row's name appears only in the test (grep from the manifest wave). Covered.
   - It reads `wantErr` against the error `prReviewClearFailure` builds in `internal/loomcli/prreviewgoto.go`: `"could not %s: %w"`, called with step `"stop the recorded PR-Review session"`. The substring matches that wording. Covered.
   - It reads the `pullrequest.Record` fields `StrandGUID`, `StrandName`, `SessionID` and `Engine` from `internal/pullrequest/record.go:35-38`. They exist. Covered.
   - It reaches `stepStopRunner` (`internal/loomcli/looparm.go:175`) through `prReviewGotoAdmitted`, which `specFor("goto")` wires at `arm.go:479`. No production code changed, so nothing downstream is forced. Covered.
2. **The record check moves to `reflect.DeepEqual` over `pullrequest.Record`.**
   - Rule checked: PATTERN-test-speed's "Reflective tests carry `//lym:seam`" (`pattern/PATTERN-test-speed.md:139-151`).
   - The tier0 scan's trigger is `NumField`, or `VisibleFields` on `reflect` (`internal/testkit/envkit/envkit_test.go:386-388`). `DeepEqual` does not walk the field set that way, so no marker is owed. Covered.
3. **The doc comment changes.**
   - The doc comment now says the hook builds its own runner and keeps the record when the stop fails.
   - Both hold against the code:
     - `stepStopRunner` builds a runner when `c.runner` is nil (`looparm.go:176-198`).
     - `clearPRReviewOnGoto` returns before `ClearReviewSession` when the stop fails (`prreviewgoto.go`).
   - The note about running without `t.Parallel` was kept, and the test still sets env through `t.Setenv`, as PATTERN-test-speed's line 190 requires. Covered.
4. **The two new imports, `reflect` and `strings`, are standard library.** No leaf-package or import rule applies. Covered.
5. **The commit message of `350370384`.**
   - Each claim holds against the code:
     - the receiver holds no runner;
     - the hook builds its own runner;
     - with no reed session up, the stop fails;
     - the record is kept whole;
     - the other rows compare the whole record.
   - That no reed session is up rests on `tmuxkit.Main`, which isolates the tmux socket for the test binary (`testmain_test.go:22`). `requireSessionLocked` then fails in `Engine.Status`, which `removeStrandIfPresent` calls (`stop.go:227`).
   - The commit carries no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer. The parent's "No bug fixed" added entry means no `Root-Cause:` trailer is owed. Covered.
6. **Declared PATTERN changes:** the directive declares none, so nothing is owed.
7. **Stencils, specs, docs and fixtures:** the diff touches none, and none describes this test's row set.
8. **`_lym/landing/summary.md`:** every named test exists under `internal/` (manifest wave), and the "No production code changes" claim matches the diff stat. Covered.

**Findings:** none.

One thing outside this lens, for the code-tests and code-correctness lenses: the `wantErr` substring also matches a failure inside `stepStopRunner` itself, when it cannot load the reed config. That failure comes out as "could not stop the recorded PR-Review session: load the shuttle and reed config…". So the row cannot tell a runner that was built and failed to stop from a runner that was never built.

**Tool calls**
1. Listed `pattern/` and grepped `PATTERN-test-speed.md` for the parallelism and marker rules: the background file was not in the manifest.
2. Read `PATTERN-test-speed.md:136-155`: the full `//lym:seam` rule, to judge the new `reflect.DeepEqual`.
3. Grepped for the `lym:seam` scanner sources: to find what counts as "reflective".
4. Grepped `internal/loomshed/seamtest.go`: it turned out to read markers, not detect reflection.
5. Grepped `internal/impactset/seams.go` for its reflection calls: it had none.
6. Grepped for `NumField`, `VisibleFields` and `DeepEqual` across `internal` and `cmd`: this found the trigger in `envkit_test.go:386-388`.
