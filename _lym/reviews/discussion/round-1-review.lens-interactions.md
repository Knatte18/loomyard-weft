```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of the record's decisions that act on the same state, and each acceptance criterion against the decisions it touches.

**Pairs checked**
- D1 × D7 (approval row inside the Motif block, `pastApproval` reads the current row): held.
- D1 × D4 (restart after Discussion approval resumes Raddle-Write; the mode stays as stored on the first call): held.
- D2 × D4 (the perch's approval is the only approval in `perch` mode; D2's auto-accept key settles a spent budget as an accept): **broken**, see F1.
- D2 × D7 (re-armed perch after a Raddle-File return reads a decision added at Raddle-Write; `pastApproval` judges by the current row, so the refusal lifts after the return): held.
- D2 × D5 (the fixer's `raddle` gate applies D5's `perch`-mode checks): held.
- D3 × D5 (the rubric names D5's checks as enforced and judges only whether one task or a family fits): held.
- D3 × D7 (the rubric reads every Added after Discussion entry, and the perch reads the record every round): held. The gap after the last approving round is the writer's stated Open risk.
- D4 × D7 (the mode is read from the rows, and the refusal is keyed on the rows): held.
- D5 × D6 (project slug equals the arras slug; the `slug-taken` exemption; `raddleFilingHeld` sees the unconverted entry as not filed, then the converted one as filed): held.
- D6 × D7 (the record file restored on a failed filing vs `copyRecordToBoard` writes; D7 refuses at Raddle-File, so there is no concurrent writer): held.
- D6 × D8 (the filing write releases no member; the converted project's empty status lets D8 release it): held.
- D6 × D10 (family dependents moved off the converted project in the same write; a run-lock refusal on a dependent returns the run to Raddle-Write): held mechanically. Whether D10's premise holds in the family case is the handler's; it is not reported here.
- D8 × D10 (the moved dependents wait on draft last parts until D8 releases them; dobby gates on `depends_on`): held.
- D5 one-task × D10 (dependents move in the filing write; `EndRaddleEntry` ends the entry done after it): held.
- D4 × D9: no shared state.

**Acceptance criteria and constraints:** all held against the decisions they touch. The D4 criterion "reaches Raddle-File on the perch's approval" says nothing about the auto-accept path, which F1 covers.

**Findings**

F1 — MEDIUM, design. Location: D2 (the auto-accept key) against D4 (Bound).
D4 says it removes the operator's approval because the perch's approval replaces it. D2 gives the split perch its own auto-accept setting, and that setting turns a spent budget into an automatic accept. Together, a proposal whose findings are still open (CONTINUE or CIRCLING) can file with no operator approval and no clean perch round. D4's Bound does not name this path, and neither decision bounds it, for example by keeping the key off in `perch` mode or by naming it a guard key the operator alone sets.
Evidence:
- `internal/shedadapters/bouncer.go:249`: "With cfg.AutoAccept set, a spent budget turns a freshly judged CONTINUE or CIRCLING round into an automatic accept"
- `internal/shedadapters/bouncer.go:1352`
- `internal/loomengine/template.yaml:44`: `discussion_auto_accept`

**Tool calls**
- `grep` of `contracts/specs/raddle-proposal-spec.md` for the `tapestry` shape: needed to check whether D5's one-task shape files under the arras slug and clashes with `EndRaddleEntry`. It does not; my context lacked the spec.
- `grep` of `internal/loomcli/decision.go` for approval detection: needed to check how D7's refusal and D2's re-arm read the current row (`pastApproval(currentRow)`). My context held only the way-forward constants.
