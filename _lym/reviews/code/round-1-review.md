---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/orch/orch-template-usage-limit-wake.md:4
    summary: "The usage-limit wake line still tells the orch to check each run with `lym batten status <slug>`, which the rewritten role forbids as polling."
    fix: "In contracts/stencils/orch/orch-template-usage-limit-wake.md line 4, replace `then check each run with `lym batten status <slug>` and restart what stopped` with wording that waits for the notices of runs that stopped and reads a status, filtered, only for a stop a notice names. The line must still render to one line, and TestRenderUsageLimitWake-style coverage must keep passing."
  - id: F2
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/landing/summary.md (Design decisions, hub rules file bullet)
    summary: "The Idea bullet that the closed #799, #849 and #885 workaround sections leave the hub rules file is not carried out by the diff, and no Added after Discussion entry records the skip."
    fix: "Leave the code as it is; do not revert anything. Defer to the run's parent, who records the skip with `lym loom decision add` (the sections sit in the Bolt or heddle rules, outside this repository's diff) or has the sections removed through `lym orch rules`. The fixer makes no change for this finding."
  - id: F3
    severity: LOW
    class: consistency
    origin: handler
    location: contracts/orchrole/orch-role.md:269
    summary: "Status reports tell the orch to give `cd <worktree> && lym reed attach`, while the new first Working rule says never cd."
    fix: "In contracts/orchrole/orch-role.md line 269, say the attach command is text for the operator to type, not a command the orch runs, for example `Every status report on a run gives its attach command, for the operator to type and never for you to run: ...`. Keep `internal/orchcli/doc.go` as it is."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: contracts/stencils/registry_test.go:21
    summary: "No test pins that the orch role stays out of the registry, so re-registering it would re-seed an editable Bolt copy with every test green."
    fix: "Add one assertion to the existing registry test in contracts/stencils/registry_test.go that Registry().Default reports no entry for `orch-template-role` and Registry().Names() lacks it. Add no new test function."
---

### [MEDIUM] Usage-limit wake line contradicts the no-polling rule

**Location:** `contracts/stencils/orch/orch-template-usage-limit-wake.md:4`
**Issue:** The role now says to watch runs through notices and never read `lym batten status` on your own to check on runs (`contracts/orchrole/orch-role.md:55`), and the entry asks to drop that watching.
The wake line the watcher types after a usage limit still says `check each run with lym batten status <slug>`, so the orch gets two opposite instructions.
The diff changed the rule but not this stencil, which states the old rule.

### [MEDIUM] Closed workaround sections: skipped without a recorded decision

**Location:** `_lym/landing/summary.md`, Design decisions
**Issue:** The entry's last Idea bullet says the closed workaround sections leave the hub rules file.
The diff cannot touch that file, which is Bolt or heddle content, and the change description explains the skip as the file being unread since `heddle-orch-target`.
That is a deviation from the entry with no `Added after Discussion` entry behind it.
Only the parent can record or refuse it, so the fix is deferred and the code stays.

### [LOW] Status-report `cd` against the new no-cd rule

**Location:** `contracts/orchrole/orch-role.md:269`
**Issue:** The command is for the operator to type, but the rule sits in a role whose first Working rule is "Never `cd`".
The Working rules section is new in this diff, so the diff introduces the apparent conflict.

### [LOW] Registry exclusion is not pinned

**Location:** `contracts/stencils/registry_test.go:21`
**Issue:** The entry's central promise is that no board copy exists.
`TestStencils_NameGoValues` and the marker rows read the role from `orchrole.Role` or `fixedStencils`, and nothing fails when `orch-template-role` is registered again alongside a restored embed.
One assertion in the existing registry test covers it.

## Rejected

- Fork finding F-B (code-affected-set): a note that `internal/orchcli/doc.go` and `rules.go` match the new text; the fork itself raised no defect.
- Unspecified hub-rule leaks into the role (the `doc.go` placement sentence, the LoomYard selfreport wording): the lines are unchanged or reworded from the previous role, and the entry does not ask to change them.
- Stale Bolt copy of the old role not removed: no tidy step for stale stencil copies exists in `internal/stencilstore` or `internal/stencilcli`, so the record's clause says to leave it, and the diff does.
- Removed `TestRenderRoleFile_RefusalsWriteNoFile` and the `role` row in `watcher_test.go`: the refusal path they covered no longer exists, and `TestStencils_TopLevelMarkers` pins the markers.
- Leftover readers of `orch-template-role`: the grep finds only `docs/research/*` snapshots, which record a point in time.
