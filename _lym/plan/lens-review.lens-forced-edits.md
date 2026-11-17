```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits — which changes force edits outside the members a card names.

**1. BLOCKING — Card 9 (delete `internal/fixtries#Write`): a test caller is not listed.**
- `internal/dobbyengine/upgrade_test.go:76` calls `fixtries.Write(dir, count)` inside `upgradeHub.deps` (`func (h *upgradeHub) deps(t *testing.T, tries map[string]int) UpgradeDeps`).
- No card lists `internal/dobbyengine#upgradeHub.deps`.
- With `Write` deleted, `TestUpgradeRead` no longer compiles. The helper has to seed `fix-tries.json` itself, for example through `os.WriteFile`.

**2. BLOCKING — Card 4: a golden file renders every shell-guard row and no card lists it.**
- `internal/shuttleengine/claudeengine#TestBuildSettings_MatchesGolden` (`settings_test.go:1105`) compares rendered settings with `testdata/settings-golden.json`.
- That file renders the existing rows: 16 `systemctl` hits.
- The new `adopt-take-hold` row changes every golden entry. Card 4 lists neither the test nor `internal/shuttleengine/claudeengine/testdata/settings-golden.json`.

**3. BLOCKING — Card 4: a test that buckets the Bash denies is not listed.**
- `internal/shuttleengine/claudeengine#TestBuildSettings_NoAgentHookAndForkGuard` (`settings_test.go:598-709`) sorts each Bash PreToolUse command by row steer. A command matching none falls into `default: bashCommands`.
- The test then wants `len(bashCommands) == 0` when `!tt.wantBashGuard`, and exactly the webster fork guard otherwise (`settings_test.go:665-672`).
- The new row's command lands in `bashCommands` and fails both branches.
- Card 4 lists only `TestBuildSettings_DenyToggleMatrix` and `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`. It needs this test as well, with an `adopt-take-hold` case and a want-exactly-one check.

**4. BLOCKING — Card 8 (`checkCmd` now runs `runHoldCheck`): an integration test is not listed.**
- Card 8 has `checkCmd` refuse with no own hold before any check runs, exiting non-zero.
- `internal/adoptcli_test#TestCheckCmd_FromABeamExitsZeroWhileOneRuns` (`adopt_integration_test.go`) expects exit 0 from a beam while a check runs, the held-lock `already_running` path. No hold is planted there, so it now meets the no-hold refusal.
- No card lists it.

**5. LOW — Card 5: orchengine's comments are left wrong.**
- Card 5's ImpactSummary says orchengine changes only comments, but no card lists orchengine.
- `internal/orchengine#OwnerWatch` field comment (`internal/orchengine/owner.go:25`): "UnderAdopt reports whether an adopt in flight on this machine covers the beam".
- `internal/orchengine/doc.go:150`: "an adopt in flight here".
- The seam now reports this machine's Bolt hold, which stands past the adopt until a green check.

**6. LOW — Card 5: the dobby's log line is left wrong.**
- `internal/dobbyengine#daemon.probeOwner` (`handover.go:36-45`) logs "adopt hold: an adopt in flight here covers this beam". Its file comment at `handover.go:2` says the same.
- Card 5 edits only `internal/dobbyengine#Deps` and `doc.go`.

**7. LOW — Card 1: boardengine's doc is listed under Uses only.**
- Card 1 changes what the replay carries, holds and registry, and refuses a whole adopt commit.
- `internal/boardengine/doc.go:195-208` describes what the replay applies, but card 1 lists `doc.go` only under `**Uses:**`.

**8. LOW — Cards 8 and 9: file header comments still describe retired behaviour.**
- `internal/adoptcli/adopt.go:1-5` names "the worker that runs them in worker.go" and "the pause adopt sets lasts until an operator lifts it".
- `internal/adoptcli/deps.go:1-4` names the origin reconcile, the adopt pause, the tries file and the adopt check start.
- `internal/adoptcli/check.go:1-3` says the check reports "the end to its nearest live orch", which `runCheck` stops posting under card 8.
- No card lists these file glyphs.

**9. LOW — Card 9: the source of `noOrch` is not stated for the `HoldCheck` seam.**
- Card 9 says `HoldCheck` runs over `hubHoldCheckDeps(prime, os.Stdout, noOrch)`.
- `internal/adoptcli#hubDepsAs` (`func hubDepsAs(prime *lymcwd.Location, machine func() (string, error)) (adoptDeps, error)`) takes no `noOrch`.
- Either `hubDepsAs` gets a re-sign arrow, or the Intent says the seam takes `noOrch` at call time.

**10. LOW — Card 1: the parse change is left open.**
- Card 1 lists `internal/boardengine#parseReplayBoard` under Edit and says the parse goes "through `parseReplayBoard` or a sibling parse".
- If `parseReplayBoard` returns the holds and registry, it needs the re-sign arrow. If a sibling does, that sibling is a `Create` handle and `parseReplayBoard` should not be listed.

Tool calls:
1. Grep for the retired wording, the `fixtries.Write` users and the re-signed functions' test callers. My context did not cover the users outside adoptcli.
2. Awk and grep for the enclosing tests of the `fixtries`, `Green` and `Failed` uses, the test package clauses, and the claudeengine test names. My context had no line-to-function mapping and no package clauses.
3. Read `upgrade_test`'s `fixtries` lines and the claudeengine fork-guard and golden tests. The subagent summary did not name those tests.
4. Checked whether the golden file holds rendered rows, which helper encloses `upgrade_test.go:76`, boardengine's doc on replay, and contract or skill mentions. My context did not cover these.
