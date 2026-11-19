```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-round change set for agreement with its surroundings (stale siblings and twins).

**Findings**

1. **LOW** — `internal/hardenshed/doc.go:73-75` (round-files paragraph, not changed by the fix). It lists the models, commits and strays files a round row writes. The fix added a fourth file, `harden-head-<round>.json`, written by a fix row at its first entry and reused on resume. The package doc is silent on it, and its "commits file … when its call returned no error" line doesn't mention that the commits file's `Head` now comes from the head file. Evidence: `round.go` `startHead`/`RoundHead`; `doc.go:74` still describes only the commits file.

2. **LOW** — `docs/overview.md:1146` and `docs/overview.md:338` (prose recipe lists). The F8 fix added `hardener` to the `Run-Shed` list at line 1953 only. Line 1146 ("a recipe declaring `contract: loom` is served, among them tapestry, motif, raddle, arras, darn, pawl and sampler") and the `shedcli` arming-table line 338 also list the recipes and still omit hardener. Line 1146 says "among them", so it is a non-exhaustive list, and I could not tell whether line 338 is exhaustive. Both are twins of the edited sentence. The review named only line 1953.

3. **LOW** — `contracts/orchrole/role-beams.md:73` ("a task run's is a tapestry, motif, darn or sampler row"). This is another orch-role-adjacent prose list of task recipes. The F8 fix edited only `orch-role.md:44`. It is not exhaustive in the same sense (it omits arras), so this is likely not stale, but it is the nearest twin.

**Checked, no finding**
- `campaign.go` `CampaignCheck.WorktreeRoot` doc is unchanged and still right: it is read for package dirs.
- `ArchiveDeps.AnchorRoot` and `Campaign.archiveDeps` agree.
- The `Campaign.Commit` doc now says "anchor-relative", matching `ArchiveDeps.Commit`.

**Calls**
1. `grep` for the `harden-*` file names in non-test Go, md and yaml: to find docs that list the round files. My context lacked the `doc.go` text.
2. `grep` for `WorktreeRoot` and `worktree-relative` in `hardenshed` non-test files: to find stale worktree-vs-anchor wording after the rename.
3. `grep` for the recipe-list phrasings in docs and contracts: to find twins of the F8 edit.
