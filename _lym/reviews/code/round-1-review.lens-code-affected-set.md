```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change in the diff, I found what reads, asserts or describes it outside the touched lines and checked whether the diff left it wrong.

**Per change, the set found:**

1. **Gated fragment item 3, the new reproduce run (`contracts/stencils/darn/darn-fragment-make-change-gated.md:15-19`).**
   - `lym gate test` argument handling, `internal/gatecli/cli.go:75,85,108`: `--` args fill `{args}`; with no `--tier`/`--gate` the call runs tier0 only. Not covered; see F1.
   - `gate.yaml` tier commands: `{args}` is present in tier0, tier1 and tier2, so `-- -run` reaches `go test`. Covered.
   - `emptyTierFailure`, `internal/gatecli/tierrun.go:404-421`: a `-run` filter exempts the no-test check, so a run that matches nothing passes. Not covered; part of F1.
   - `tiergate.Require`, `internal/testkit/tiergate/tiergate.go:70-71`: a test of another tier skips. Part of F1.
   - Cold-fixer hand-off, `internal/gatecli/cardfixer.go:77-90`: only a `--gate <key>` call whose key has a `fixer` hands off. The stencil's package-plus-`-run` form names no key, so no fixer touches the reproduce run. Covered.
   - Rule `PATTERN-test-speed` ("A site or stencil names a gate key, never a tier list, scope form, test command or target list", `pattern/PATTERN-test-speed.md:226`). Not covered; see F2.
   - Template Step 3, `contracts/stencils/darn/darn-template-write.md:43-60`: it exists and has the outdated-entry stop. The pointer resolves. Covered.
   - Gate rule in the system prompt, `internal/shellguard/rows.go:374,419`: package form `lym gate test ./internal/x` with `--tier`. The stencil defers to it. Covered, except the tier; see F1.
2. **`DarnInputs.GateFixed` field comment and `DarnTable` doc (`internal/darnengine/spec.go:67-68,79-81`).**
   - `internal/darnengine/doc.go:78-83`: updated. Covered.
   - `contracts/stencils/stencils.go:53-55`: updated. Covered.
   - Fragment header comment, lines 1-3: updated. Covered.
   - `internal/loomcli/darnwiring.go:279` ("has the writer's prompt drop its gate runs"): not updated; see F3.
3. **The pinned Step 3 string (`"3. Run no \`lym gate test\` yourself."` becomes `"…yourself, bar one run."`).**
   - `internal/darnengine/spec_test.go:76`: updated. Covered.
   - `internal/loomcli/wiring_darn_test.go:266`: updated. Covered.
   - Repo-wide grep: no other holder of the old string. Covered.
   - `wiring_darn_test.go:262` subtest name ("the writer's prompt drops its gate runs only for …"): not updated; see F3.
4. **Docs and rule documents describing the gated writer.**
   - `docs/overview.md:1032` and `pattern/PATTERN-gate-slot.md:83-104` describe Darn-Write's gate rows, not the writer's own runs. Nothing stale.
   - `docs/research/loom/darn-writer-anatomy-2026-11-08/report.md:175` and `docs/research/turns/turn-savings-measure-2026-11-04/report.md:39` are measurement and research reports, exempt as point-in-time records.
5. **Commit messages and departures.**
   - `b4ef3eb4`'s body matches the diff.
   - `78fcaeaa` is a comment-wrap fix.
   - No `Pattern-Departure:` trailer exists.
   - No PATTERN changes are declared, so nothing to land.

**Findings:**

- **F1 — BLOCKING, design.**
  - Location: `contracts/stencils/darn/darn-fragment-make-change-gated.md:16-18`.
  - Summary: the reproduce-run instruction gives no tier, so for any test above tier0 the run reports a false pass. The stencil then tells the writer to treat the entry as overtaken.
  - Evidence:
    - `internal/gatecli/cli.go:75` ("--tier N … the default is tier0 without --upto or --gate").
    - `internal/testkit/tiergate/tiergate.go:70-71`: `t.Skipf` for another tier.
    - `internal/gatecli/tierrun.go:404-421`: a `-run` filter exempts `emptyTierFailure`, so a filter matching nothing, or a renamed test, also passes silently.
  - The issue's own case was a shed driver-loop test, likely not tier0.
  - Fix direction: tell the writer to pass `--tier <n>` from the test's `tiergate.Require` line, and to confirm the run reports the named test ran before reading a pass as "no longer reproduces".
- **F2 — BLOCKING, design. A rule conflict for the handler to judge.**
  - Location: `contracts/stencils/darn/darn-fragment-make-change-gated.md:17`.
  - Summary: the stencil now tells the agent to build a scope (its package) and test-command args (`-- -run <TestName>`). `PATTERN-test-speed` says "A site or stencil names a gate key, never a tier list, scope form, test command or target list" (`pattern/PATTERN-test-speed.md:226`).
  - The conflict: a `--gate <key>` call hands a red verdict to a cold fixer when the key carries a `fixer` (`internal/gatecli/cardfixer.go:77-90`). That would let a fixer change code around the very failure the writer is checking.
  - The constraint wins by the prompt's rule. Either the stencil's form changes, or the PATTERN entry needs an exception declared and landed.
- **F3 — LOW, consistency.**
  - Location: `internal/loomcli/darnwiring.go:279` and `internal/loomcli/wiring_darn_test.go:262`.
  - Summary: the wiring comment ("has the writer's prompt drop its gate runs") and the subtest name ("drops its gate runs") still say the gated prompt drops all gate runs. The diff updated the sibling comments in `spec.go`, `doc.go` and `stencils.go`, and `TestDarnSpec`'s case name, to name the one allowed run.
  - Evidence: `darnwiring.go:279` and `wiring_darn_test.go:262`, read in context; `wiring_darn_test.go:266` was edited in the same subtest.

**Tool calls:**
1. Repo-wide grep for "run no gate" and "drop its gate runs" phrasing: the manifest's grep covered only the exact old Step 3 string.
2. Grep of deployed specs and `docs/` for darn and gated-fragment mentions: docs and specs were not in the manifest.
3. Grep of `docs/overview.md`, the glossary, `PATTERN.md` and the gate-slot background for darn gate text: to check that rule and overview docs describe the writer's runs correctly.
4. `tiergate.Require` and the gate's no-test summary: the manifest lacked how a test of another tier behaves under the default tier.
5. `internal/gatecli/tierrun.go:400-449` (`emptyTierFailure`): to confirm whether a `-run` filtered run that matches nothing or only skips is reported or passes.
