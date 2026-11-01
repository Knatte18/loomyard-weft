```yaml
lens: code-affected-set
outcome: reported
```

**Affected-set list (code-affected-set lens)**

1. **Registry removal (`OrchTemplateRole`, the `entries` row).**
   - Set found: Go and stencil references to `orch-template-role`, `OrchTemplateRole` and `roleStencilName`. After the diff, none remain outside historical `docs/research/*` reports, which are measurement records.
   - Covered. The scan tests (`citation_enforcement_test.go`, `govalues_test.go`, `markers_test.go`, `parentdirective_test.go`, `roleskills_test.go`) were moved to `shippedStencils`, so the role is still scanned.

2. **`RenderRoleFile(stencilsDir, path, index)` became `RenderRoleFile(path, index)`.**
   - Set found: `internal/orchcli/start.go:580`, `internal/orchengine/resumecontext.go:27`, `internal/orchengine/watcher.go:193` and `prompt_test.go:36`.
   - Covered. All four were updated and no other caller exists.

3. **Deleted refusal for a missing marker** (`TestRenderRoleFile_RefusalsWriteNoFile` and the `role` row in `watcher_test.go`).
   - The shipped role's markers are pinned by `markers_test.go`, and `TestRenderFiles_WriteWithoutLeadingComment` checks the filled output. Covered.

4. **New seam import `contracts/orchrole` in `orchengine`.**
   - `seam_enforcement_test.go` allowlist updated. Covered.

5. **Role text, "no status polling".**
   - Set found: `internal/orchcli/doc.go:77` was updated. I found one stencil that contradicts the role's new "never read `lym batten status` on your own" rule:
     - `contracts/stencils/orch/orch-template-usage-limit-wake.md:4` tells the orch to "check each run with `lym batten status <slug>`". It is not covered.

6. **PATTERN docs.**
   - `PATTERN.md:59` and `pattern/PATTERN-stencil-ownership.md:14` agree: both name the orch role, `contracts/orchrole`, no board copy, and heddle's `orch-prompt`. Covered.
   - No Pattern-Departure trailer is needed, and there is none.

7. **Stale Bolt copy tidy clause.**
   - The record says to remove a stale Bolt copy only "if a tidy step for stale stencil copies exists".
   - I saw no such step in the diff. The skip looks correct, but I did not read the stencil seeding or tidy code, so I could not confirm that no such step exists.

**Findings**

- **F-A: MEDIUM, consistency.**
  - Location: `contracts/stencils/orch/orch-template-usage-limit-wake.md:4`.
  - Summary: the usage-limit wake line still tells the orch to "check each run with `lym batten status <slug>`" after `Run lym tender revive`. The rewritten role (`contracts/orchrole/orch-role.md:55`) forbids reading `lym batten status` on your own to check on runs, and the entry says to drop that watching.
  - Evidence: the file line above; `orch-role.md:55`.
  - Fix: reword the stencil to wait for the notices of runs that stopped, with a status read only for a stop a notice names.
  - Rubric: "Stencil wording" and "Scope against the answer key".

- **F-B: LOW, consistency.**
  - Location: `internal/orchcli/doc.go:61` and `internal/orchcli/rules.go:21`/`:38`.
  - Summary: these describe the orch rules and the heddle path. They match the new role text (`orch-role.md:308`–`313`), so I raise no finding on them.
  - Not a finding, noted only to show they were checked.

- **Out of lens, flagged for the handler:** the diff touches no hub-rules file, so the entry's "closed workaround sections leave the hub rules file" is not in the diff. That file lives on Bolt or in heddle, not in this repository's tree.

**Tool calls**
1. `grep` for "role stencil", "batten status" and similar across the orch stencils, `orchengine`, `orchcli`, `stencilstore`, `heddleengine`, the README and `docs`. My context lacked the stencils other than the role, and the docs.
2. `grep` for `orchrole` and for `orch-template` seeding or list references. I needed to confirm that every importer and every registry-style reader was updated.
