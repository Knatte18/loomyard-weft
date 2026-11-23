```yaml
lens: code-scope
outcome: reported
```

Lens: code-scope. I traced each change in the diff to the board entry's O1–O5 goal points and to the parent's added O3 entry.

**Per change: the decision it traces to**

- **C1.** `RefuseHubWithoutBolt` (`clone.go`) and its call in `runReconcile`, placed before the Bolt pull (`fabriccli/fabric.go:652`). Traces to O1.
- **C2.** `existingHubError` and `cloneResetCommand`, used at `CloneHub`'s three "hub already exists" sites. Traces to O1. The entry names `:225` and `:264`. The third site, at `:371`, gives the same refusal in the one-argument form, so it belongs to the same change.
- **C3.** `reconcile_missingbolt_integration_test.go`. Traces to O1, which asks for this regression test.
- **C4.** The clone paragraphs added to `fabriccli/doc.go` and `fabricengine/doc.go`. Forced by C1 and C2, since the docs had to describe the new refusals.
- **C5.** The `resolveMergeSources` layout read and the new error return in `merge.go`'s `mergeIn` and `Merge`. Traces to O2's fix, which allows "return it as the verb's refusal".
- **C6.** The layout read in `fastForwardWeft` (`parenttips.go:410`). Traces to O2's sentence on the same gap: "`pull.go:186` and `recordsgit.go:346` via `FastForwardPair`". The entry names that path, so this is not unrequested scope.
- **C7.** The layout reads in `FetchPairTips` and `ReturnToPreviousTransition` (`codetipsettle.go`). Traces to O2, which names "`codetipsettle.go:21,43` via `DobbyRuns`".
- **C8.** `TestOneRepositoryMerge_RefusesARecordsStoreThatLostItsFetchRefspecs`. Traces to O2.
- **C9.** The remove and cleanup `--help` text, plus the corrected `DropLocal` doc (`destroy.go:2241`). Traces to O3 and the parent's added entry. `archive.go` (`pushLocalArchiveTags`) is untouched, as that entry requires.
- **C10.** `PlainArchiveTag`, `PlainArchiveTagsOnOrigin`, the cleanup `plain_archive_tags` field, the help and doc lines, and `TestOneRepositoryOrigin_PlainArchiveTagsAreListedWithTheirMove`. Traces to O4.
- **C11.** O5: the `Rerun` field on the job and the pending record, the new `rerun` parameter of `DiscardTaskBranch`, and `discardRerunWayForward(job)`.
  - The battencli callers and the updated test call sites are forced by the signature change.
  - Naming the command in the held-lock refusal (`lockRerun`) goes past the entry's "job and the pending record". It serves O5's purpose, though, which is printing a command the operator can type, so it stays in O5's scope.
- **C12.** Commits `a990e79f6` and `8dcbb5725`, which only re-break comment lines. Forced by C1–C11's comments; no scope of their own.

**Per decision: the changes that build it**

- O1: C1, C2, C3 and C4.
- O2: C5, C6, C7 and C8. The `bolt.go` path (`lym machine name`) is left unchanged, and that holds: `Bolt.Pull` already reads the layout before it fetches (`bolt.go:305` before `:319`).
- O3 and the added entry: C9.
- O4: C10.
- O5: C11.

**Findings:** none. Every change traces to a decision or is forced by one, and every decision has a change that builds it.

One thing outside this lens, for the handler: `_lym/landing/summary.md` says the O1 refusal includes `--single-repo` for a one-repository hub. `RefuseHubWithoutBolt` builds that case as `CloneOptions{WeftURL: …}` with no `SingleRepo` (`clone.go`, diff 003 line 125), and the test expects no `--single-repo`.

Tool calls:
1. Grepped `internal/fabricengine/bolt.go` for its layout read and fetch, and ran `git diff --stat` over the range. The manifest read only lines 150–170 of `bolt.go`, so it couldn't show whether the untouched `lym machine name` path already reads the layout.
