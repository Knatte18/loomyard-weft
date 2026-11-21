## Goal

```markdown
{"error":"invalid json: invalid character 'w' looking for beginning of value","ok":false}

## Triage against main (2026-11-21)

- Create check: `internal/planglyph/donecheck.go:108` reports create-not-done without reading the decision record; `createTargetWayForward` in `internal/websterengine/recordbatch.go` gives the two verbs different ways forward.
- Drift halt: `internal/websterengine/verifygate.go` cuts findings at `verifyGateStuckTextBytes` (400 bytes) and names no report file; nothing lets the parent accept expected drift.
- Plan amendment: `internal/burlerengine/roundcheck.go` refuses a finding in a file the changes file does not list, with no exemption for an `--amends-plan` decision, and Webster's halt reason never reaches Plan-Burler-Fix as a finding.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-21): Scope after the corrupted board body

Decision: The board body was lost in a merge (orch error). Scope is the darn agent's three-part proposal from issues #168, #170, #171, #172 and #178: (1) the Create check reads the decision record, a Create a decision drops reports as dropped-by-decision and does not block, and record-batch and recover-batch give the same way forward; (2) the drift halt names a full report file instead of cutting findings at 400 bytes, and its way forward names how the parent accepts expected drift; (3) an --amends-plan decision exempts the cards it names from the changes-file hunk rule in burler's round check, and Webster's halt reason reaches Plan-Burler-Fix as a finding. Also in scope: #178 part 1, the batch fork's prompt marks a later added decision as overriding an earlier one it supersedes. Out of scope: the auto rebaseline after a fix (#170 bullet 2).

Rationale: The issues are the source of the lost notes. #178 part 1 is the defect that cost a halt on single-repo-namespace-layout, so it stays in.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
