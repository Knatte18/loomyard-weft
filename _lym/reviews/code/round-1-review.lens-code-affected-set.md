```yaml
lens: code-affected-set
outcome: reported
```

Task: list what the diff's changes force outside the lines it touches, per change, and report any member left wrong.

**Per-change set**

| Change | Set found | Covered |
|---|---|---|
| `OwnerRequest.Replace` and the replace-set write in `SetOwners` | `boardengine/doc.go` owner-write paragraph; the commit-subject text in `doc.go` and in `ownerset.go`; `pattern/PATTERN-owner-machine.md` clause "takes a project or the prime ... when `lym machine name`'s sweep names that recorded owner"; `PATTERN.md` line 24; the only production caller, `machinecli/set.go:195`; the `TestSetOwners` rows | Covered. The `PATTERN.md` overview text already equals the declared text and needs no edit. |
| New `machine` module | `cmd/lym/main.go` import, `AddCommand` and "Available modules" string; `cmd/lym/sandbox_coverage_test.go` exclusion with a Why; `cmd/lym/seamsignature_test.go` for `RunCLI` and `RunCLIIn`; `docs/overview.md` package tree and module list; `pattern/PATTERN-cli-cobra.md` Deviations (no engine package) | Covered. I found no other module-name list in a non-research file that omits `machine`. Only `internal/orchcli/rules.go` and `internal/configreg` name `heddle`, and neither is a registry that applies to an operator-audience verb. |
| `machinename.FromThisHost` | `machinename.go` doc and implementation; `TestFromThisHost`; the PATTERN-owner-machine "sole-source scan", which flags a hostname read outside the package, and the call is inside the package | Covered |
| `machine.json` error text and its way forward (`lym machine name <name>`) | `machinename.go` `setNameWayForward`; `beamcli/dobby_test.go:134-144` already names `lym machine name` | Covered. The verb the existing messages point to now exists. |
| Declared PATTERN change (`PATTERN-owner-machine`) | The declared text equals the current `PATTERN.md` line 24 and the background file's line 3. The background clause gained the sweep case. | Landed. Nothing further is declared. |
| Commit messages and departures | Commits `f9273aa2`, `94851c91`, `dee9e0e2`, `92c2747d`, `4b63dcd6` and the merge, as shown in the `git log` already read | No `Pattern-Departure:` trailer appears in the log, and none is needed. |

**Findings**

1. LOW, consistency: `pattern/PATTERN-owner-machine.md:40-42`. The decision record's Constraints say the entry's background "gains that the name comes from the machine file before the hostname." The background does carry that, and I checked it against the file. This is not a defect.
2. LOW, consistency: `internal/boardengine/ownerset.go:34-35`. The `OwnerRequest` doc still reads "records Machine as the owner of the root project and of beam projects, or claims one entry for it." It does not mention the replace-set mode, although `Replace` has its own field comment and `doc.go` covers the mode. This is an optional tidy-up, not a rule violation.

I found no BLOCKING or MEDIUM finding for this lens.

**Tool calls (1 of 6)**
1. `grep` for module-list names (`heddle`) and for `machinecli` or `lym machine` across the repo. My context lacked a search for other module registries and other references to the new verb.
