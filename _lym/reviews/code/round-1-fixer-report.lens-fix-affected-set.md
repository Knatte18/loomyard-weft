```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set lens over F1 (verbs and dispatch) and F2 (`ResolveWithin` branches), one search each.

**F1.** The search `grep -rn ResolveWithin` finds one production caller, `internal/shuttlecli/cli.go:89`. `cmd.Name() == awaitForksVerb` is the only dispatch site, and the new `TestRunCLI_AwaitForksResolvesFromTheRecordsSibling` covers it with one `await-forks` row and one `state` row. The set is covered. Other strict verbs (`run`, `open`) get no row, but they share the same `lymcwd.Resolve` default and the review asked for one other verb.

**F2.** `ownerOfRecordsSibling` has four exits:
- A name without the suffix is covered by the existing rows.
- A suffixed name whose owner directory is absent is covered by the new `SuffixedNameWithNoOwnerBesideIt`.
- A suffixed name whose owner exists is covered by the existing records-sibling rows.
- The `owner == ""` and owner-is-not-a-directory exits have no row, but the review did not name them.

The no-marker row uses `root`, which has no `.lym-anchor` marker and is resolved from a subdirectory, so it covers the `anchorRelOrRoot` fallback. The new `lone-weft` row fails if the existence check is removed, because the owner would then resolve as `lone` and not `lone-weft`.

Findings:
- LOW. Changed text: the header comment at `internal/lymcwd/lymcwd_test.go:4`, which lists what the file covers for `ResolveWithin`. It does not mention the two fallbacks the new `TestResolveWithin_NoOwnerOrNoMarker` pins (a suffixed name with no owner, and no marker). The sibling summary is now stale. Evidence: line 4 reads "ResolveWithin's anchor-subtree and records-sibling acceptance and the stale-marker refusal".

Findings left unchecked: none.

Tool calls made:
- One `grep -rn "ResolveWithin"` across the repo's Go files, to enumerate the callers and tests of the resolver, since my context held only the two files read, not the whole repo.
