```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens on the diff at 16490acdd..HEAD.

**Per change, the set found and whether it is covered**

1. **New `tmux-script` row and `SwitchAutonomous` in `internal/shellguard/rows.go`**
   - Package doc `internal/shellguard/doc.go:64-67`: covered.
   - The init check that every switch is known exists in both engines: `claudeengine/settings.go:645-652` and `codexengine/guards.go:73-85`. Both maps gained the switch, so this is covered.
   - Manifest guard candidates walk every row: `claudeengine/manifest.go:306` and `codexengine/manifest.go:266`. Covered.
   - `pattern/PATTERN-refusal-way-forward.md`, the pattern-shaped list: covered.
   - The `PATTERN.md` entry `PATTERN-shell-guard-table` says every engine renders every row. Both engines map the switch, so this is covered.
   - Golden `settings-golden.json`: regenerated, covered.
   - `cmd/lym/norecoverydeny_test.go` reads the table's forms, and the row has none. Nothing is forced.
   - `contracts/orchrole/orch-role.md:107` names `tmux capture-pane`. The orch is interactive (`orchcli/start.go:228`), so it is not affected.
   - The sampler stencil's `tmux -L` line is for an interactive spec (`samplerengine/spec.go:90`), so it is not affected.
2. **The steer's way forward, "in the form your instructions give for a single test"**
   - It relies on a stencil form for every autonomous role. Only the two darn make-change fragments give a single-test form. No burler reviewer, driver, burler/webster/landing fixer or cold-fixer stencil gives one. **Not covered**, see F1.
3. **Engine wiring**
   - Claude: `denyInputs.interactive` feeds both maps. Covered, and `claudeengine/doc.go:260` is updated.
   - Codex: the one production `guardInputs` literal is at `codexengine.go:180`. Covered.
   - Codex `doc.go` does not describe guard rows, so nothing is forced.
4. **Gated darn fragment rewritten from one exception to two**
   - The `internal/darnengine/spec.go:88-89` DarnTable doc comment still says there is one exception. **Stale**, see F3.
   - The `GateFixed` field doc (`spec.go:72-74`) still holds.
   - The stencil's own header comment still holds.
   - The `darnengine/spec_test.go:146` assertion on "run that one test class once through" still matches the text. Covered.
5. **Ungated darn fragment, new lines.** Its form "`lym gate test` with its package and a filter" lacks `--tier`. A call with no tier flag runs tier0 only (`internal/gatecli/doc.go:29`), so a tmux-tier test is skipped. **Wrong**, see F2.
6. **Cold-fixer frame, new section.** The `contracts/stencils/markers_test.go:80` marker list is unchanged by the edit. Covered.
7. **Commit messages and trailers**
   - There is no `Pattern-Departure:` trailer.
   - The `Root-Cause:` trailer on a7937a620 names the missing table row, which the diff adds. It is consistent.
   - The directive declares no PATTERN changes, so that bullet raises nothing.

**Findings**

- **F1.** Severity MEDIUM, class `design`, location `internal/shellguard/rows.go:390` (`steerTmuxScriptDeny`).
  - The steer sends every autonomous role to "the form your instructions give for a single test", but only the darn writer fragments give one. Reviewers, shed drivers, and burler, webster, landing and cold fixers get a way forward that points at no instruction.
  - The steer also never says `--tier`, which board item 1 asks the refusal to name ("run it through `lym gate test --tier`").
  - Evidence: rows.go:390, the decision record's Fix item 1, and `contracts/stencils/gatefix/gatefix-frame-cold.md`'s new section, which gives no form. A grep for "one test" or "single test" across `contracts/stencils` hits only the darn fragments and the loom plan-review rubric.
- **F2.** Severity MEDIUM, class `design`, location `contracts/stencils/darn/darn-fragment-make-change.md` (the new lines after Step 3's `refuse_raw` sentences).
  - The ungated writer is told to run its tmux-tier test through "`lym gate test` with its package and a filter that selects it", without `--tier`.
  - With no tier flag the gate runs tier0 only, so a test that requires the tmux tier is skipped. The writer would read a fact from a run that never ran the test.
  - Evidence: `internal/gatecli/doc.go:29` ("the default is tier0 when neither `--tier` nor `--upto` is given") and the hub `gate.yaml`, where tier2 holds the tmux tests.
- **F3.** Severity LOW, class `consistency`, location `internal/darnengine/spec.go:89`.
  - The DarnTable doc comment says "The one exception is the writer's single measuring run of one test class for a value only a test run shows."
  - The gated fragment now has two exceptions, the second a fact about an external tool's behaviour, so the comment is stale.
  - Evidence: spec.go:88-89 against the gated fragment's new "There are two exceptions."

Tool calls:
1. Grepped `PATTERN.md`, skills and contracts for tmux/script commands, stencil marker tests, "one exception" texts and the shellguard doc. My context lacked the PATTERN.md entry, the skill and stencil hits, and the darnengine doc's wording.
2. Read `internal/darnengine/spec.go:72-92`, the darnengine/gatefixer doc grep and orch-role's tmux line, to confirm the stale exception text and the orch's tmux use.
