---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md (Interview Turn 4, Turn 6 pattern-fit bullet, Question ledger Q7)
    summary: The support log still describes the title form as pinned to the orch stencil by a test and claims a PATTERN-producer-pointer-rule departure named in Constraints, which the decision record no longer says
    fix: >-
      In support-log.md, change Turn 4's closing clause "declared in `internal/alignment` and pinned to the orch stencil by a test" to "declared in `internal/alignment` and rendered into the orch role stencil through a template key"; change the Turn 6 bullet "pattern-fit, `PATTERN-producer-pointer-rule` departure: fixed, named in Constraints." to "pattern-fit, `PATTERN-producer-pointer-rule` departure: fixed, the stencil renders the title form through a template key, so no second literal copy exists and no departure remains."; and in Q7 replace "and the decision-note title form has one Go declaration and one stencil site that a test pins together, so no third site exists for an entry to guard" with "and the decision-note title form has one Go declaration that the orch role stencil renders through a required template key, so no second copy exists for an entry to guard".
---

### [LOW] Support log describes the superseded test-pinned stencil design

**Location:** `_lyx/discussion/support-log.md`, Interview Turn 4, the Turn 6 `pattern-fit, PATTERN-producer-pointer-rule departure` bullet, and Question ledger Q7.

**Issue:** D3 now renders the title form into the orch role stencil through a template key that joins `RenderRoleFile`'s required markers, so the form has one declaration and no literal stencil copy.
The support log still records the earlier design in three places: Turn 4 says the form is "pinned to the orch stencil by a test", Q7 justifies "no new PATTERN entry" by "one Go declaration and one stencil site that a test pins together", and Turn 6 says the `PATTERN-producer-pointer-rule` departure was "fixed, named in Constraints", but Constraints names no such departure.
The decision record is right and the plan writer never reads the support log, so nothing the plan writer gets is wrong.
The discussion review does read the support log, though, and these lines misstate what was decided and why Q7 holds.
Q7's conclusion still holds under the template-key design; only its reason is stale.

## Rejected

- Focus 1, D3's bound: not a defect.
  D3's Bound says what the narrowed guard now lets through: free-form notes, existing free-form notes for a held task once this lands, and an operator-written note without the `Decision for <slug>:` title such as one the bouncer escalation stencil counts by its body, so an approve given while such a note is added is applied.
  It names the bound: only the title form counts, and the operator's ways to hold an approve are that title or `lyx loom decision add` when a record exists.
  That meets the attack-surface rule.
  The arbiter's body-based definition (`contracts/stencils/bouncer/bouncer-template-escalation.md` line 37) differs on purpose, and Scope Out covers it.
- Focus 2, single declaration through a template key: not a defect.
  D3 declares the form once in `internal/alignment` as a function of the slug, renders it into `orch-template-role` through a key filled for the literal `<slug>`, and adds the key to the required markers that `internal/orchengine/prompt.go` `RenderRoleFile` already checks for `command_index` and `repository_rules`, with the same restore-or-delete way forward.
  `go list` shows `internal/alignment` does not depend on `internal/orchengine`, so the key adds no import cycle, and no PATTERN entry is broken: `PATTERN-stencil-ownership` keeps the stencil read at call time, and `PATTERN-producer-pointer-rule` is met because the stencil copies nothing.
  The only stale part is the support log's account, which is F1.
- Focus 3, which runs reach the zero bound: not a defect.
  The Goal limits the guarantee to "a run whose seed carries its start stamp", and D1's Bound lists what reaches the zero bound: a seed written before `started_at` existed, a removed seed or a hand-edited stamp.
  It also says why a reused slug does not: batten's completed-slug refusal (`internal/battencli/arm.go`) requires `lyx batten cleanup --apply`, which removes the run directory, and the loom and darn bootstraps (`internal/loomcli/sharedbootstrap.go`, `internal/battencli/wire.go`) then write a fresh seed that `WriteSeed` stamps.
  I checked this against `internal/shedrun/seed.go`: `WriteSeed` returns early on an agreeing seed without restamping, and `ReadSeed` returns not-found with no error for a missing file and an error for a read or decode fault, which is the split D1 relies on.
- Focus 4, D2's `way forward:` clause: not a defect.
  D2 and its acceptance criterion both name "close that pull request on GitHub, then resume the run with `lyx loom start`".
  That matches the resume verb both push-rejected reasons in `internal/landingshed/publish.go` already end with.
- Support log `## Review rounds` still reads `_No rounds yet._` after round 1: not raised.
  By spec, that section's round entries come from the review gate, not the writer.
