```yaml
lens: code-scope
outcome: reported
```

Task: trace the diff's changes to the board entry's Fix items 1–3 (code-scope lens).

**Per change**

- **C1** adds the `tmux-script` row, `SwitchAutonomous`, its steer and notice, and the doc.go text in `internal/shellguard/rows.go:24-29,388-398,790-805` and `internal/shellguard/doc.go:64-67`. It traces to Fix item 1.
- **C2** wires `SwitchAutonomous` into both engines: `claudeengine/settings.go:378,388`, `codexengine/guards.go:29,50,60`, `codexengine/codexengine.go:180` and `claudeengine/doc.go:260`. It traces to Fix item 1 ("every engine").
- **C3** adds the `tmux-script` rows to `rows_integration_test.go`. It is forced by C1.
- **C4** updates the engine tests (claude `manifest_test`, `settings_test`; codex `guards_test`, `manifest_test`, `prepare_test`, `usersetup_test`) and `testdata/settings-golden.json`. It is forced by C1 and C2.
- **C5** updates `pattern/PATTERN-refusal-way-forward.md:33`, which now lists the tmux/script deny as pattern-shaped. It is forced by C1, because the rule document went stale.
- **C6** adds the second exception to the gated writer fragment, plus the narrowness sentence and the no-background rule (`darn-fragment-make-change-gated.md:24-34`). It traces to Fix items 2 and 3.
- **C7** adds the tool-learning and no-background lines to the ungated writer fragment (`darn-fragment-make-change.md:17-20`). It traces to Fix items 3 and 2.
- **C8** adds a `## External tools and processes` section to `gatefix-frame-cold.md:30-33`. It traces to Fix item 3.
- No change lacks a trace.

**Per decision** (the record has no `Added after Discussion` entries)

- **Fix item 1** (refuse `script` and raw `tmux` in every run agent's Bash, and name the way forward): built by C1 and C2, but only in part. See F1 and F2.
- **Fix item 2** (a run agent can run one tmux-tier test, narrow enough that it cannot start the whole tier): built by C6 and C7, in writer stencil prose only. See F3 and F4.
- **Fix item 3** (writer and fixer stencils say to learn tools from a gated test and to start no background process): built by C6, C7 and C8, for the darn writer and the cold gate fixer only. See F4.

**Findings**

- **F1 — MEDIUM, class `decision`**, at `internal/shuttleengine/claudeengine/settings.go:378` and `codexengine/codexengine.go:180`.
  - Summary: the row is installed only when `!spec.Interactive`, but item 1 asks for it in every run agent's Bash.
  - Run sessions that can launch interactive keep raw `tmux` and `script`: the Discussion seat (`loomengine/discussiontable.go:173`, `cfg.DiscussionInteractive`), the PR session (`landingshed/prsession.go:215`, `Interactive: !autonomous`), the drift judge (`driftengine/judge.go:193`), the raddle (`loomengine/raddletable.go:122`) and the seat chair (`seatengine/prompt.go:250`).
  - The summary's "Autonomous runs only" decision is the writer's own call. The record holds no Discussion and no added entry for it.
  - Fix: leave the code as it is. Defer to the parent, who records or refuses the narrowing with `lym loom decision add`.
- **F2 — MEDIUM, class `scope`**, at `internal/shellguard/rows.go:390` (`steerTmuxScriptDeny`).
  - Summary: item 1 says the refusal names the way forward as "run it through `lym gate test --tier`", but the steer names `lym gate test` with a package and a filter and never names `--tier`.
  - The gate's default is tier0 (`internal/gatecli/doc.go:29`). On this hub the tmux tests are tier2 (`_bolt/_lym/config/gate.yaml`). A call in the steer's form therefore runs no tmux-tier test, unless the agent's own instructions supply the form.
  - The same gap is in the ungated writer stencil (`darn-fragment-make-change.md:18`).
- **F3 — LOW, class `decision`**, at `contracts/stencils/darn/darn-fragment-make-change-gated.md:30`.
  - Summary: item 2 leaves "how narrow that opening is" to a Discussion, but none was held. The writer settled it in prose ("The call names one package or project and selects one class"), and nothing enforces it.
  - An agent can still run `lym gate test --tier 2 --all`. The summary admits that no gate-side check was added.
  - Fix: defer to the parent, who records the narrowness decision.
- **F4 — MEDIUM, class `scope`**: Fix items 2 and 3 are not built for the other writer and fixer stencils.
  - Only the darn writer fragments and the cold gate fixer frame changed.
  - None of these untouched stencils carries the tool-learning or no-background rule: `burler/burler-template-fixer.md`, `webster/webster-fragment-card-fixer.md`, `landing/landing-template-fixer.md`, `twill/twill-writer-fork.md`, `twill/twill-writer-fork-gated.md`, `loom/loom-template-part-writer.md`, `seat/seat-directive-writer.md` and `webster/webster-template-plan-fix.md`.
  - Of those, only `webster-template-plan-fix.md:22` says anything near it ("never run a Bash call in the background").
  - The cold fixer frame (`gatefix-frame-cold.md:32`) tells the fixer to learn from a test the gate runs, but gives it no single-test form to run one. So item 2's opening does not reach fixers.
  - Fix: add item 3's two rules to each writer and fixer stencil that edits code, and give each of them the single-test gate form.

Tool calls:
1. Grepped the untouched writer and fixer stencils for the new rule and for gate wording. The manifest listed their names but not their contents.
