---
verdict: BLOCKING
prior:
  - id: F1
    status: resolved
    answer: steerTmuxScriptDeny now names --tier, the package that holds the test and a filter after --, and the claudeengine golden carries the new text in every steer and notice copy with no trace of the old clause.
  - id: F2
    status: resolved
    answer: darn-fragment-make-change.md Step 3 runs the test through lym gate test --tier <that tier> with its package and a filter after --, and has the writer check the report shows it ran.
  - id: F3
    status: resolved
    answer: The burler, landing, weaver batch and webster implementer and verify-fix fork stencils carry both rules with the --tier single-test form; the conflict resolver and arbiter fix carry the no-background rule. The card fixer fragment renders into webster-body-implementer.md, so it gets that section.
  - id: F4
    status: resolved
    answer: "Settled by the added entry \"F4: tmux/script refusal reaches interactive run sessions too\"; the diff does not yet follow that ruling, raised as N1."
  - id: F5
    status: resolved
    answer: "Settled by the added entry \"F5: lym gate test refuses a run agent's whole tmux tier\"; the diff does not yet follow that ruling, raised as N2."
  - id: F6
    status: resolved
    answer: The DarnTable doc now says the single measuring run serves a value only a test run shows or a fact about how an external tool behaves, matching the gated fragment's two exceptions.
findings:
  - id: N1
    severity: BLOCKING
    class: design
    location: internal/shellguard/rows.go:804
    summary: "The added entry F4 rules that the tmux-script row is installed in every run session, interactive ones included, with the sampler exempt through a named switch on the spawn spec and not through Interactive; the row still hangs on SwitchAutonomous, which claudeengine/settings.go:378 and :388 and codexengine/codexengine.go:180 map from !spec.Interactive, so the Discussion seat, PR session, interactive drift judge, raddle and seat chair keep raw tmux and script"
    fix: "Add a named bool field to shuttleengine.Spec (internal/shuttleengine/spec.go), such as AllowRawTmux, documented as the sampler's exemption from the tmux and script deny, and set it in the sampler's spec at internal/samplerengine/spec.go:90. Replace SwitchAutonomous in internal/shellguard/rows.go with a switch named for that exemption (such as SwitchDenyRawTmux), installed and announced when the spec does not set the field; map it in claudeengine settings.go shellSwitchInstalled and shellSwitchAnnounced from a denyInputs field fed by the new spec field, and in codexengine guards.go switchInstalled and switchAnnounced from a guardInputs field fed the same way in codexengine.go Prepare, never from Interactive. Drop \"in an autonomous run\" from steerTmuxScriptDeny and reword the row comment, the SwitchAutonomous doc, internal/shellguard/doc.go and claudeengine/doc.go to say the row reaches every run session but the sampler. Update the engine tests (claudeengine manifest_test.go, settings_test.go; codexengine guards_test.go, manifest_test.go, prepare_test.go, usersetup_test.go) so an interactive spec gets the row and only a spec setting the exemption lacks it, regenerate claudeengine/testdata/settings-golden.json, and correct the Autonomous-runs-only claim in _lym/landing/summary.md."
  - id: N2
    severity: BLOCKING
    class: design
    location: internal/shellguard/rows.go:390
    summary: "The added entry F5 rules that lym gate test refuses a run agent's tmux-tier run naming no package target and no test filter, with the single-test form as its way forward; no file under internal/gatecli changed, so the whole tmux tier is still open to a run agent and the steer's way forward points at a form nothing enforces, while _lym/landing/summary.md says no gate-side check was added"
    fix: "In internal/gatecli, where the run request is checked before any slot (the refusal checks in run.go beside the --map and --copies refusals), refuse a call from a run agent whose tiers include the hub's tmux tier when it names no package target and no filter after --; read the run-agent condition from the session environment lym already sets for a strand, through its owning package. The refusal follows PATTERN-refusal-way-forward: it names the way forward as lym gate test --tier <the tmux tier> <package> -- <a filter selecting one test>, in the same terms as steerTmuxScriptDeny, and a test in internal/gatecli reaches the refusal and asserts the way forward text. Document the refusal in internal/gatecli/doc.go, and replace the No-gate-side-check design decision in _lym/landing/summary.md with the check."
---

Prior findings F1, F2, F3 and F6 hold as fixed.
F1's steer, the golden's eight steer and notice copies, and `doc.go` all name `--tier`, the package and a filter after `--`.
The codex texts derive from the same row.
F2's ungated Step 3 now names `--tier`.
F3's stencils carry both rules.
The remaining stencils that name `lym gate test` without the no-background rule are review rubrics, the plan-card block, and `gatefix-body-card.md`, which renders inside `gatefix-frame-cold.md`, so none of them leaves a code-editing agent without the rule.
F6's doc comment agrees with the gated fragment's two exceptions.

The parent ruled on F4 and F5 after round 1, and both rulings go against what the diff does.
Each held finding is resolved as a question, and the gap between the ruling and the code is raised as a new BLOCKING finding.

### [BLOCKING] The tmux and script row skips interactive run sessions, against decision F4

**Location:** `internal/shellguard/rows.go:804`

**Issue:** The added entry "F4: tmux/script refusal reaches interactive run sessions too" requires the row in every run session but the sampler, exempted by a named switch on the spawn spec, "not a check of Interactive".
The row hangs on `SwitchAutonomous`.
`claudeengine/settings.go:378` and `:388` map it from `!in.interactive`.
`codexengine/codexengine.go` maps it from `!spec.Interactive`.
So every interactive run session keeps raw `tmux` and `script`, the exact narrowing the parent refused.
The steer's "in an autonomous run" and the switch's doc carry the same narrowing.

### [BLOCKING] lym gate test still lets a run agent start the whole tmux tier, against decision F5

**Location:** `internal/shellguard/rows.go:390`

**Issue:** The added entry "F5: lym gate test refuses a run agent's whole tmux tier" asks for a check in the gate, saying that prose in the stencils alone is not enough.
The range touches nothing under `internal/gatecli`, and the change description states that no gate-side check was added.
The narrowing still rests on the stencil sentence "The call names one package or project and selects one class, so it never starts a whole tier", which nothing enforces.
The steer at this line names the single-test form as the way forward with no gate refusal behind it.
The finding sits here because the incremental rule keeps a new finding on a changed hunk of a listed file.
The fix site is `internal/gatecli`, named in the fix.
