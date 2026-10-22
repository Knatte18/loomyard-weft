---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: contracts/stencils/orch/orch-template-role.md:176
    summary: the stencil says lyx config profiles lists the hub's profiles, but that command is the editor and refuses without a terminal
    fix: In contracts/stencils/orch/orch-template-role.md line 176, replace "`lyx config profiles` lists the hub's profiles." with "`lyx board profiles` lists the hub's profiles." The orch reads the list with that verb; `lyx config profiles` only edits them.
    origin: handler
  - id: F2
    severity: LOW
    class: scope
    location: internal/runprofile/runprofile_test.go:39
    summary: the new darn block in the template's light profile has no test that pins it
    fix: In internal/runprofile/runprofile_test.go, extend the existing test that loads the template's starting profiles (the one asserting the names budget, docs, light at line 39) with an assertion that the light profile holds darn {review_max_bounces 1, review_hard_max_bounces 2}. Add it as an assertion in that test, not as a new test.
    origin: lens:code-tests
  - id: F3
    severity: NIT
    class: consistency
    location: contracts/stencils/orch/orch-template-role.md:175
    summary: the new bullet joins two independent clauses with a semicolon on one line, against the repo's semantic-line-break rule for .md files
    fix: In contracts/stencils/orch/orch-template-role.md line 175, end the line after "before its batten run starts;" and put "it never gets hand-written `run_settings` for its review bounds." on the next line, indented as a continuation of the same bullet.
    origin: lens:code-pattern-fit
---

### [MEDIUM] Stencil names the wrong command for listing profiles

**Location:** contracts/stencils/orch/orch-template-role.md:176
**Issue:** `lyx config profiles` opens the profiles in an editor; without a terminal it refuses (`internal/configcli/doc.go`), and the orch has none. The verb that lists the hub's profiles is `lyx board profiles` (`internal/boardcli/cli.go:347`, whose help text says "Edit the profiles with `lyx config profiles`"). An orch following the new rule would run the wrong command when it looks for a profile name.

### [LOW] No test pins the light profile's darn block

**Location:** internal/runprofile/runprofile_test.go:39
**Issue:** The diff adds `darn: {review_max_bounces: 1, review_hard_max_bounces: 2}` to the seeded `light` profile. The runprofile tests check only the starting profile names, and the `Lookup("light")` test reads a hand-written file, so a regression of the block would pass.

### [NIT] Semicolon clause not broken onto its own line

**Location:** contracts/stencils/orch/orch-template-role.md:175
**Issue:** CLAUDE.md requires one sentence per line and a break at an independent-clause boundary in every `.md` file; the new bullet keeps both clauses on one line.

## Rejected

- Stale hard cap of 4 left in other places: rejected. The remaining `review_hard_max_bounces: 4` pins in `internal/loomcli/wiring_test.go` and `internal/loomengine/template.yaml` belong to loom, whose default the entry does not change. The `hardCap: 4` fixtures in `circling_test.go` are hand-set fakes, not the darn template.
- Loomyard hub's own `darn.yaml` not touched by the diff: not a defect. The file lives on Bolt outside the worktree, and the summary says so; the board entry's removal step is a post-landing hub config edit.
- Dropped rationale (perch fixer self-check, cap of 2 buying one more round) from the stencil: not a defect, since the entry asks for the rule to be dropped.
- `light.darn` validity under the profile schema: held. `decode` in `internal/runprofile/runprofile.go` accepts the same shape the `budget` profile already uses, and the values 1 and 2 satisfy the hard-cap rule.
