```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, the set of things outside its lines that must change with it, and whether the diff covers them.

**Per-change set**

| Change | Set found | Covered |
|---|---|---|
| New `createDeps.ClearMotifRun` field | `hubCreateDeps` (`internal/beamcli/deps.go:63`) and the test world (`create_test.go:109`) are the only `createDeps{` constructors, and the other constructor at `deps.go:83` was not checked against this field | Yes |
| New `createResult.MotifRunCleared` and key `motif_run_cleared` | `internal/beamcli/doc.go:26` and the struct comment. Neither `contracts/orchrole/orch-role.md` nor `docs/overview.md:456` describes the envelope keys | Yes |
| New durable `child-recipe` file in the run directory | `shedrun/doc.go`, `battenRunCommitPaths` and its test, and the `removeRunRecords` removal, since `os.RemoveAll(RunDir)` takes it with the directory. It is plain text, so `PATTERN-tolerant-records` does not apply, and its path comes through `lymdirs`, so `PATTERN-durable-vs-ephemeral-state` holds | Yes |
| `WriteSeed` closure now writes the record | Its direct caller `seedChildProducer.Call` (`battenshed/seamchild.go:93`) maps an error from it to a hard error, not Stuck. `battencli/wire_test.go:437-470` only exercises the refusal paths | Yes, for callers. Whether a test pins the new write is the tests lens's question |
| New `battencli.ClearMotifRun`, an extra removal path | `battencli/cli.go:12` and the `cleanup.go` header are updated. `doneSlugRefusal` (`arm.go:878`) and its test (`cli_test.go:278`) stay valid because other done runs are still refused | Yes |
| Behaviour: `lym beam create` now clears a done motif run | `docs/overview.md:456` and `docs/overview.md:692` | No, see finding 1 |
| Commit messages and PATTERN | No `Pattern-Departure:` trailer in the range. My prompt declares no PATTERN changes, so nothing is owed there | Yes |

**Findings**

1. LOW, class `consistency`. `docs/overview.md:456` and `docs/overview.md:692` are not updated.
   - Line 456 is the beam summary and says nothing of the new clearing step.
   - Line 692 describes `lym batten cleanup` as the only way a done run's records go.
   - The diff updates `beamcli/doc.go`, `shedrun/doc.go` and `battencli/cli.go`, so the package docs land.
   - The rubric says docs the repository names for this kind of change land in the same diff. I found no document that names `docs/overview.md` for it, so I rate this LOW.
   - Evidence: `docs/overview.md:456`, `docs/overview.md:692`; the diff touches no `docs/` file.

No other member of the affected set is left wrong.

**Tool calls (3 of 6)**
1. A repo-wide grep for the new symbols, `createDeps{` constructors, `motif_run_cleared` and `child-recipe`. My context lacked the other users and docs of the new names.
2. A grep for sibling per-run record handling (`SettleRecords`, `battenRunCommitPaths`) and for the "has already completed" refusal text, plus the `arm.go` refusal source. My context lacked the other enumerators of run-directory files and the refusal's definition.
3. Reads of `contracts/orchrole/orch-role.md`, `docs/overview.md` and `docs/` mentions of `beam create` and cleanup. My context lacked the docs outside the diff that describe these verbs.
