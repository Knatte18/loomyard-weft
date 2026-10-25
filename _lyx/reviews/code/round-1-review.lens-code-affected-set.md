```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens — for each change, who calls, reads or asserts it, and is each of those left correct.

**List (per change: set found, covered or not)**
- Retired `parent-review` and `lyx loom review` names. I searched code, stencils, docs and PATTERN outside `_lyx`. The only leftover is `internal/creelengine/doc.go:202`. The `reportengine` hits are the D4 read-only parse and are licensed. Covered, except F1 below.
- Retired config keys (`alignment`, `alignment_timeout_min`, `skip_alignment`, `parent_review_wait_min`). `loomengine/config.go` drops them from `Config`, `ConfigSpec`, the validation and `retiredLoomKeys`. The drop reports `<key>: retired, removed`. No remaining reader turned up in my grep. Covered.
- New escalation cause `decision`. I found these readers:
  - `EscalationCause.valid` and the three error strings in `shedadapters/escalation.go` and `circling.go` list four causes. Covered.
  - The `lyx loom circling` help text and refusal text in `loomcli/circling.go:11,43,275,284` name the decision cause. Covered.
  - `arbiterstop` refuses it through `DecisionHalt`. Covered.
  - `arbiterstop.go:89` admits only hard-cap and circling awaiting escalations to the arbiter, so a `decision` escalation falls through to the refusal. Covered.
  - The brief stencil has the new cause bullet and the optional `cause_detail` marker. Covered.
  - The orch role stencil carries the `decision` cause at `orch-template-role.md:51`. Covered.
- `DecisionHaltPrefix` halt. Callers of `arbiterstop.HandsBack` are `loomcli/arm.go:312`, `loomcli/drivewiring.go:319` and `battencli/wire.go:973`, and each now includes the decision halt. Covered.
- `BurlerFix` `DecisionsEscalated`. The Bouncer key is `escalate_decisions` and the fix row key is `decisions_escalated`. I did not check how the Env or recipe layer passes the fix-row key through to `BurlerHalfDeps`, because I had no calls left to trace it. Not verified.
- `Merge-In` → `Parent-Align`. The loom recipe, `landingshed` and `reportengine` carry the new name or keep the old one where D11 requires it. The remaining `Merge-In` and `merge-in` hits are fabric's own merge-in, git-level prose, or the intended retired-name handling. Covered.
- `merge_fix_attempts` → `parent_align_fix_attempts`. The migration is in `landingshed/config.go:194`. No other reader turned up. Covered.
- Lenses `goal-scope` and `board-fit`. Both are rewritten as procedures and `review-discussion` gains them. `twill-discussion` membership is unchanged. Covered. I did not re-count the read calls, since the text states six.

**Findings**

- **F1, LOW, consistency.** `internal/creelengine/doc.go:202` still says `the parent-review gate posts to the parent itself`. That gate no longer exists. The sentence is a stage description for the `creel-agents` task, which the record leaves out of scope (Scope Out). D2 says no text names parent review for the step between Discussion-Write and Discussion-Review. Fix: reword the line so it does not name a removed gate, or note that it records a historical stage. Evidence: `internal/creelengine/doc.go:202`.

I found no other member of an affected set left wrong in what I could check.

**Tool calls**
1. A grep for leftover retired names, case-insensitive, across non-test files. I needed it because the manifest only covered a sample of the diff.
2. A grep of `circling.go` and the escalation-cause references. I needed it to check that every reader of the cause set was updated.
3. A grep for the `HandsBack` and `DecisionHaltPrefix` callers, the orch stencil mentions of `decision`, and the `creel-agents` line in `creelengine/doc.go`. I needed it to find the consumers of the new halt.
