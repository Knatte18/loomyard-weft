# Card 6 — Pin agent-filed issues and update the reflection stencil and docs

**Create:**
- `cmd/lyx/agentfiledissues_test.go`

**Edit:**
- `contracts/stencils/claims_test.go`
- `internal/friction/doc.go`

**Prosa:**
- `PATTERN.md`
- `pattern/PATTERN-friction-capture.md`
- `docs/overview.md`
- `contracts/stencils/friction/friction-template-reflection.md`

**Intent:**
The reflection agent decides whether an anomaly is worth an issue (decision D5), and a test keeps Go code from acquiring a filing path again (decision D6).

- `cmd/lyx/agentfiledissues_test.go`: a `scankit.Walk` scan, shaped like `cmd/lyx/ghguard_test.go`, failing when a non-test `.go` file outside `internal/selfreportcli/` and `internal/selfreportengine/` contains `selfreportengine.CreateIssue`; the allowlist uses `scankit.NewAllowlist` with `RequireNoStale`, and the failure names `PATTERN-agent-filed-issues`.
  Before card 1 this test would have failed on `internal/loomcli/cli.go`.
- `PATTERN.md`: add `PATTERN-agent-filed-issues` after `PATTERN-github-auth`: filing a GitHub issue from lyx goes only through `lyx selfreport create`, run by an agent or the operator; no other production package calls `selfreportengine.CreateIssue`. (test)
  One line, no background file, within `internal/pattern`'s entry-line cap.
- `pattern/PATTERN-friction-capture.md`: the first bullet also covers the `loom-crash-resume` note written at a `run` or `step` entry; replace the `selfreport`-knob bullet with: Go-detected anomalies become friction notes, a halt note carries its anomaly kind and history rows, and `lyx loom start` vouches for the resume it spawns; rewrite the Enforcement paragraph to name `halt_test.go` and `entryobservation_test.go` instead of `selfreport_test.go` and the pending retry.
- `internal/friction/doc.go`: "Why the leaf exists" no longer says Tier 1 lives in `anomaly.go` and files issues; the Go-authored notes section names the halt, crash-resume and refusal notes.
- `docs/overview.md`: the `selfreport` module line keeps manual filing through `lyx selfreport create` and says that agents (the friction reflection) are its only automatic caller; drop the Tier-1 detection, pending retry and `selfreport` key text.
- `contracts/stencils/friction/friction-template-reflection.md`: add `loom-crash-resume` notes to Step 1's Go-written kinds, and say halt notes name their anomaly kind; delete the Tier 1 sentence and the "halt note without that line" sentence in Step 2.
  Add the rule: for a `loom-halt` or `loom-crash-resume` note, file an issue only when the notes, the reason or the trace show a lyx problem behind the event; a halt caused by the task's own code, or a review segment that did not converge, is dropped; a process signal such as a spent budget or an escalation is not a defect.
  What the rule lets through and what bounds it (decision D5): when the agent misreads a halt, a real lyx fault can stay unfiled, but the evidence is not lost — the note names the trace file, the run's status and history stay on disk, and the operator can still file through `lyx selfreport create`.
  The stencil states the drop rule next to that pointer, so the agent knows a dropped event stays recoverable, and the `PATTERN-friction-capture` bullet records the same bound.
  Step 3 says every issue body states what happened, why it is abnormal and what to do, then the existing provenance line, and that the label follows `lyx selfreport create`'s own `--label` guidance (point at the verb's contract, never restate it, `PATTERN-producer-pointer-rule`).
  Keep the `{{.parent_directive}}` marker and every other marker.
- `contracts/stencils/claims_test.go`: replace the "do not file the halt event" claim with claims for `loom-crash-resume`, the lyx-problem-only filing rule, and the what-happened/why-abnormal/what-to-do body, plus a `mustNot` on `Tier 1`.

Run `grep -rn -i "tier 1 anomaly\|selfreport: \|selfreport.go\|filed-title" --include=*.md --include=*.go .` afterwards; only `crucible/` and `_lyx/` hits may remain.

**ImpactSummary:** prose and one new scan test; the reflection's filing rule changes for halt and crash-resume notes only.

**Commit:** 6: Pin agent-filed issues; reflection stencil and docs follow the friction-note anomalies
