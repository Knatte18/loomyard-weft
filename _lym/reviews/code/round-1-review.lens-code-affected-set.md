```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, I looked for what calls, reads or asserts it outside the touched lines.

**Per change: set found, covered or not**

1. **Guard replaced by `refuseRunSettingsCaller` in `runsettings.go`.**
   - The old refusal text, "refused from the task session … way forward: the operator or the hub orch makes the change", no longer appears in any test, doc, stencil or fixture. The only remaining matches are `internal/configcli/hubwide.go:49` and `internal/fabriccli/roleguard.go:59`, and those belong to other guards. Covered.
   - Three call sites pass through the guard, all via `prepareRunSettings`, which has a fixed signature: `cli.go:540`, `cli.go:606` (batch upsert) and `cli.go:931`. All reach the guard with `fields["slug"]` and `fields["project"]` as the guard reads them. Covered.
   - Only the single upsert verb is tested at the boardcli level. The batch path is not exercised by the new beam-orch test, but it runs the same function, so I raise no finding.
   - `battencli`'s `prepareRunSettings` is a separate function and is unaffected. Covered.
2. **`Board.BeamUnder` added in `parentbeam.go`.**
   - The only production caller is the guard. Docs in `boardengine/doc.go:109` and `boardcli/doc.go:68` name it. Test: `parentbeam_test.go`. Covered.
   - There is no PATTERN entry to update. `PATTERN-owner-machine` and `PATTERN-pair-parent` govern how a parent is picked for an owner read and for a new pair. The guard's use of `BeamUnder` is a permission check and does not read an owner. `ParentBeamOf`'s seam is `hubgeom.ParentOf`, the one resolver, and `BeamUnder` goes through it. No violation.
3. **Doc updates in `boardcli/doc.go` and `boardengine/doc.go`.**
   - Both match the code's behaviour. The "both as stored and after the write" wording and the way-forward text agree with the guard's refusal string.
4. **Orch-role stencil, `contracts/orchrole/orch-role.md:218`.**
   - The diff does not touch it. It already tells an orch to name a profile in `profile` before the batten run starts, and it is not scoped to the hub orch. So the Goal's first sentence is already met, and I raise no finding.
5. **`configcli/hubwide.go:48` still refuses on `agentname.TaskSlug`.**
   - It is a separate guard on hub-wide config writes, and the Goal does not license changing it. It is the only other production `TaskSlug` caller, and it is intentionally unchanged. Not a finding.
6. **Commit messages and PATTERN.**
   - There is no `Pattern-Departure:` trailer and none is needed. The directive declares no PATTERN changes, and the diff touches no `PATTERN.md` or `pattern/` file.

**Findings:** none.

**Tool calls**
1. grep for the old and new refusal text across the repo, to find asserting tests, docs and fixtures that my context lacked.
2. grep of contracts, pattern, docs and the board code for `TaskSlug` and "task session" mentions of the guard.
3. Read `orch-role.md` lines 210–225 to see whether the stencil already carries the profile instruction.
4. grep for callers of `prepareRunSettings`, `strandNameFromEnv`, `BeamUnder` and `ParentBeamOf` outside tests.
5. Read `cli.go` lines 594–613 to confirm the batch path passes the payload map with its `slug` and `project` fields.
