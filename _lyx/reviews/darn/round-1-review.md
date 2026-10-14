---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: contracts/stencils/twill/twill-directive.md:25
    summary: a writer relaunched before its Agent call may not hold the reviewed files in context, yet its forks are told never to read them again
    fix: In the relaunch paragraph that begins "A writer relaunched after its process died before its Agent call", add that the writer reads back every reviewed file it did not write in this session before it runs the procedure, so the forks inherit them. Update the matching claim in contracts/stencils/claims_test.go (the "died before its Agent call still spawns once" row) only if the sentence it pins is reworded.
---

### [LOW] Relaunched writer may spawn forks that cannot see the files

**Location:** `contracts/stencils/twill/twill-directive.md:25`
**Issue:** The fork boilerplate says the forks review the files "as your context holds them, and never read them again".
That holds for a writer that wrote the files in its own session.
A writer relaunched after its process died before its Agent call starts a new session whose context holds none of what the dead process wrote.
The relaunch paragraph tells it to "finish writing" and self-check, but not to read back files an earlier process already wrote.
Its forks then inherit a context without those files and are barred from reading them, leaving only the 6-call allowance for something "your lens needs that your context lacks", which a whole plan directory exceeds.
The board entry asks that a pre-spawn death still spawns once; it does not say how the forks get the files.
The gap is narrow because it applies to a relaunch only.

## Review notes

The rest of the diff matches the board entry and the summary.

- The directive reads the rubric, spawns, waits, judges and records after the writing steps, with the relaunch rules the entry asks for, and the fork boilerplate drops the ready-marker wait and keeps the 6-call cap and report line.
- The spawn points in `internal/loomengine/twill.go` name the files-written-and-self-check-passed condition for both hosts.
- A repository-wide search for the ready marker, `ReadyMarker`, `WriteScope`, `twill-write-scope`, `twill_write_scope`, `PrepareDiscussionSpawn` and `PreparePlanSpawn` finds no remaining reader; the only hits are the unrelated ready markers of shuttle, battenshed and the shuttle await verb, which the summary says stays.
- DiscussionWrite's fresh-spawn preparation is `PrepareRound` alone behind the parent-review gate and nil otherwise, and PlanWrite's is the rotator alone, as the summary claims.
- The Discussion template's former `{{.twill_write_scope}}` line is now a literal blank line, and a `single` or `seats` prompt has the same bytes, since an empty optional marker also rendered a blank line.
- `internal/twilldirective` doc, the `discussion.go` and `plan.go` file docs and `docs/overview.md` match the new behavior; no new invariant needs a PATTERN entry, and the diff only removes tests, so no test-budget edit is owed.
- The removed `PrepareDiscussionSpawn` and `PreparePlanSpawn` tests covered only seams that no longer exist; the fresh-spawn and attach behavior of the remaining preparations stays covered by the existing entry tests, and the changed claims and `TestDiscussionSpec_Twill` pin the forks-after wording without duplicating each other.
