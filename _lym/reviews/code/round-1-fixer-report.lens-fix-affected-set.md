```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the set searches behind F2 (callers and copies of the parent-merge filter), F1 (sites of the scratch-record override) and F5 (orch-role siblings). One finding.

**Findings**

- LOW. Changed text: `internal/landingshed/finalize_pr.go:194` (F1 fix). The package doc `internal/landingshed/doc.go:267` still says "an open pull request in the run's own record is merged as it stands". That already matches the new open-only behaviour, so this is fine. The sentence "and otherwise the resolver decides" also still holds, since a closed or merged scratch record now falls through to the resolver. No stale text. Not a finding.
- LOW. Changed text: `internal/landingshed/finalize_pr.go:308` (F1 fix). The merged-lacking re-resolve in `recordedBeamLanding` passes `BeamWithoutPullRequest` with `found`, then calls `landingForDecision` with no open-scratch check. This is a different path that the review never named: the beam's own landing record holds a pull request, so the scratch override never applied there. I raise it only as a possible twin site; it is not a defect from the fix.

**Searches by finding**

- **F2:** The filter that skips records-only commits and merges whose second parent the parent branch holds now has one copy, `BeamCommitsPast` at `internal/fabricengine/beamline.go:114`.
  - `internal/fabricengine/taskpatch.go:132` also reads `Parents[1]`, but for a task patch tip, so it is unrelated.
  - The other `CommitLog` callers are `websterengine`, `peers.go:320` and the task-run seam at `internal/loomcli/landingdeps.go:387`. Those are task or webster uses, which the review said to leave alone.
  - The pawl-only `CommitsMissing` override is the one wiring site for a pawl run. Task runs keep `CommitLog`.
- **F1:** The `BeamWithoutPullRequest` sites are `finalize_pr.go`, `beamcli/land.go:238` and `beamcli/pr.go:357`. The last two only call the resolver, so there is no other scratch-record override.
- **F5:** The "Parent-Align and Finalize fix their own failures" and "no pull request waits" wording exists only in `contracts/orchrole/orch-role.md`, and that file is now scoped to task runs. The other `Parent-Align` hits are in specs and docs about task runs and need no change.
- **F3, F4 and F7:** These are single-site coverage-gap or local changes, not sets. I did not search them.

**Calls**
1. `grep` for `Parents[1]` and `CommitLog(` across non-test Go code, to find other copies of the parent-merge filter and other `CommitLog` callers. My context had no such search.
2. `grep` for `Parent-Align` and the orch-role phrases in contracts, docs and orchengine, plus `BeamWithoutPullRequest` across non-test Go code, to find siblings of the F5 text and the F1 override sites.
3. `grep` for "as it stands" in docs and md files, to find prose describing the scratch-record override. My context had not read `internal/landingshed/doc.go`.
