```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Per change, the set I found and whether it is covered:**

- **New `parent-review` group and verbs.** Covered: `inReviewGroup` and its doc comment, the `resolvePersistentPreRun` doc comment, `parent.AddCommand`, `docs/overview.md` (verb list and config prose), `internal/loomcli/doc.go` and `internal/burlerengine/doc.go`. Not covered: the pre-run skip test (F1 below). I found no shellguard, settings or allow-list entry naming `decision add` or `peers`, so nothing else needs the new verbs.
- **New keys `discussion_review_ask_parent` and `discussion_review_parent_timeout_min`.** Covered: `config.go` (struct, `ConfigSpec` map, validation), `template.yaml` (both comments say "settable per run through run_settings"), `config_test.go` (`templateDefaults`, override row, refusal rows) and `docs/overview.md`. The only other files naming `pr_review_ask_parent`, which I used as the sibling-key search, are `landingdeps.go`, `orch-role.md` and the PR-review stencil. They concern PR-Review and are unaffected. No `hubreconcile` change is needed, since no existing key's meaning changes.
- **`lymdirs.ParentReviewDirName`.** Covered: the constant is named in `lymdirs` and used by `burlerengine`. The lock comes from `lock.TrackedLockPath`. Nothing else enumerates `lymdirs` names.
- **`chairTimeout` extraction.** Covered: `CapAssistTimeout` is the only other caller, and the `TestCapAssistTimeout` rows are unchanged.
- **Declared PATTERN changes and `Pattern-Departure:` trailers.** None are declared, and none of the five commit messages carries a trailer. Nothing to check.
- **Stencils.** The diff changes none. The chair stencil is deferred to `discussion-review-parent-seat`, and I found no existing stencil text that the new verbs contradict.

**Findings**

- **F1, LOW, `consistency`.** `TestResolvePersistentPreRun_SkipsSlugGroups` does not cover the new group.
  - Location: `internal/loomcli/cli_test.go:40`, and its comment at `:29`.
  - Evidence: `inReviewGroup` now skips `parentReviewGroup` (`internal/loomcli/cli.go:290`). Commit `f0760833` says the loom pre-run skips the parent-review group. The test still iterates `[]string{"circling", decisionGroup, peersGroup}` and its comment lists only "circling, decision and peers groups".
  - Effect: removing `parentReviewGroup` from `inReviewGroup` would arm the group's slug as a run-id and break both verbs, with no test failing.
  - Fix: add `parentReviewGroup` to the slice and update the comment to name the parent-review group.

**Tool calls**
1. Grep for `pr_review_ask_parent`, `decision add|peers show` and `lymdirs` names repo-wide: to find the sibling-key, verb-list and directory-name sets.
2. Grep of `orch-role.md`, the PR-review stencil, `PATTERN-lym-files.md`, `PATTERN.md` and the shellguard/settings dirs: to check rule text and allow-lists.
3. Grep for `decisionGroup|peersGroup` outside the loomcli files: to find group-enumerating code and tests.
4. Read of `cli_test.go:20-70`: to confirm that test's group list.
