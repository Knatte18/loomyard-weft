```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I checked everything outside the changed lines that the diff's changes reach.

**Per change, the set found and whether it is covered**

1. **`refuseUnseededBoard` message and way forward** (`pushclassified.go:184-245`).
   - Caller: `pushClassified` at `pushclassified.go:132`, reached through `Bolt.Push` at `bolt.go:146`. Covered: the error is returned unchanged and no failed-push record is written.
   - Package doc: `doc.go:522-528` is updated with the three origin states. Covered.
   - Tests: `BoardWithNoUpstreamIsNeverPushed` in `clone_singlerepo_integration_test.go:579`. Covered.
   - No production code, spec, stencil, doc or skill quotes or parses the old text ("so it was never seeded onto origin", "has no upstream in a one-repository"). I searched for it. Covered.
   - `docs/getting-started.md:35` and the `lym fabric clone` usage line (`fabriccli/clone.go:214`) only name the `--reset` flag. Nothing there goes stale.
   - The upstream settings the refusal prints match what the seed writes: `remote=origin` and `merge=ns.Branch(branch)` at `destroy.go:1793-1805`. The test fixes the same values at `clone_singlerepo_integration_test.go:117-119`. Covered.
   - `PrimeRecordsBranchKey` is written at `clone.go:391` and is read the same way. Covered.

2. **`withdrawUnansweredSeedPush` now returns `(withdrawn bool, err error)`** (`destroy.go:1838`).
   - Its only callers are the two in `pushCloneSeed` (`destroy.go:1758`, `destroy.go:1779`), and both are updated. No other reference exists outside `destroy.go`. Covered.

3. **`pushCloneSeed` wording of a withdrawn seed push.**
   - Doc comment: `destroy.go:1717-1718`. Covered.
   - Package doc: `doc.go:540`. Covered.
   - Tests: the `BoardLanded` and `RecordsBranchLanded` rows at `clone_singlerepo_killedpush_integration_test.go:72-91`. Covered.
   - Nothing else matches these message strings. The `seedPushCause` prefixes these tests assert are unchanged. Covered.

4. **Repository rules.**
   - `PATTERN-refusal-way-forward` binds only webster, shed and loom (`pattern/PATTERN-refusal-way-forward.md:3`). `fabricengine` is not bound, so the entry does not apply to these refusals. No denied form (`rm -rf`, `git push --force`, `git reset --hard`) appears in the new text.
   - `PATTERN-fabric-boundary` holds: every git read stays in `fabricengine` through `gitexec` and `gitrepo`.
   - `PATTERN-code-conventions` holds: the branch and the namespace come from the named constants `OneRepositoryBoardBranch`, `RecordsNamespaceRoot` and `ns.Branch`, and `originRemoteName` is used for the remote.
   - The directive declares no PATTERN changes, so there is nothing to land.

5. **Commit messages and departures.**
   - Commit c22357bcc carries `Root-Cause:` naming the inference from the missing local upstream. That is the defect the `refuseUnseededBoard` change removes.
   - No `Pattern-Departure:` or `Loosened-Check:` trailer appears in the range.
   - f7d565cb6 and a32d529bd describe their own hunks: comment wrapping and the unborn-board branch, and the comment of the seeded-origin row.

**Findings**

None. Every member of the affected set I found is covered by the diff.

**Tool calls**
1. Searched for the old refusal text and the changed symbols across the repo, outside `.lym` and `_lym`, to find stale readers. My context held only the definitions.
2. Searched contracts, docs, cmd, pattern and plugins for docs that pair `--reset` with `--single-repo`. The manifest had not covered docs or specs.
3. Read the upstream and seed mentions in `bolt.go`, the `--reset` lines in `fabriccli/clone.go` and `docs/getting-started.md` to check for stale descriptions. These files were outside the manifest.
4. Searched Go code for matches on the refusal's text, to find any parser of the message.
