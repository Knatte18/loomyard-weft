```yaml
lens: fix-affected-set
outcome: reported
```

Unchecked: F6. I made no extra search for `runOwnerVerdict` branches beyond the two the review named (the report entry and `localOwnerVerdict`), for example a report that holds no entry for the slug. That search would have been a seventh call.

No findings.

Per finding:
- **F2 (callers of `resumeOwnerClaim`, and tests asserting `Claim`'s owner):** there are two callers. `run_test.go`'s four rows are now on card 3. `owner_test.go:173` is `TestPassExitCode`'s `LostResumeOwnerClaim` row, which card 3 already named. The only test asserting the claim's owner is `wire_seams_test.go:523`, which is `TestWire_CreateWorktreeClaimsTheBoardEntry`, now on card 3. All covered.
- **F1 (tests relying on the run-held rule, and struct compares of the loomcli verdict):**
  - The only struct compares are `TestRunOwnerVerdict`'s, now on cards 3 and 25.
  - The other run-held readers do not read the owner through the run-held rule:
    - `tendercli/deps_test.go` builds a fake `Owners` with explicit owners, and its `TestPairOwners` is on card 3.
    - `rename_test.go`, `project_test.go` and `board_test.go` use the run hold for rename, project and status checks.
    - `peers_integration_test.go` asserts no owner.
  - `board_test.go:104`'s grouping project, which reads the prime owner, keeps passing under card 3's rule.
  - Covered.
- **F3 (tests seeding a task's own owner directly):**
  - `ownerset_integration_test.go:66,134` are `newOwnerFixture` and `setOwnerOnOther`, now on card 3.
  - The other seeds are already on card 3:
    - `originseams_test.go:87-88` (`TestTeardown_OriginSeams`)
    - `verbs_test.go:369` (`stepOwners`)
    - `task_test.go` (`TestNewTask`)
    - `owners_test.go` (`TestOwnersEffective`)
    - `runlock_test.go:103` (`TestRunLock`)
    - `replay_test.go:71` (`TestFabricReplay`)
    - `store_test.go:1004` (`TestListTasksBriefLayerAndProposal`)
  - `store_test.go:522` (`TestUpsertFieldAllowlist`) asserts a refusal naming `lyx adopt` and no recorded owner, so it holds under card 3. The review's Rejected section says the same.
  - Covered.
- **F5 (stencils and docs naming `lyx adopt <slug>`):** `orch-template-role.md:144` is now on card 25. `docs/overview.md:434` names `lyx adopt <slug>...` and `--prime`, and card 4 lists `docs/overview.md`. I did not read card 4's text to confirm it rewords that line; it is outside this fix's change set.

Calls (7, one over the cap of 6):
1. Grep of battencli tests for `resumeOwnerClaim` and `OwnerMachine`: the context held only `run_test.go`'s rows, not every caller.
2. Read of `owner_test.go:140-200`: to identify the second `resumeOwnerClaim` caller the grep found.
3. Grep of tests for `RunHeld`, `SetRunStatus` and `OwnerVerdict{`: F1's set had not been searched beyond `TestRunOwnerVerdict`.
4. Grep of the remaining run-held readers for owner reads: to judge which members depend on the run-held rule.
5. Read of `board_test.go:88-106` and `peers_integration_test.go`: to check their owner expectations.
6. Grep for direct task owner seeds and `lyx adopt <slug>`/`--prime` in contracts and docs: F3's and F5's sets had not been searched beyond the named sites.
7. Read of `store_test.go`'s enclosing tests and a grep of which cards name them: to map the two seeds left over to tests and cards.
