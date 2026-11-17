```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: checked the F1/F2 change set against its sibling docs and twin call sites. Four findings, none BLOCKING.

1. **MEDIUM — `RecordsBranchLive` is the twin of the `Add` change and still uses the current prefix only.**
   - Changed text: `add.go` now sets `warpBranch` from `PairTaskBranchOrOrigin` under `AdoptOnly`.
   - Evidence: `remoteleftover.go:84` still builds `RecordsBranchName(t.cfg.BranchPrefix + slug)`. Its doc says it reports whether the weft branch is live "by the test Add adopts a pair's weft branch by". The only non-test caller is `boardcli/cli.go:82`.
   - For a pair made under `a/` and read on a `b/` machine, `Add` now checks `a/<slug>-weft` while `RecordsBranchLive` checks `b/<slug>-weft`. The two disagree.

2. **MEDIUM — The `Add` doc comment is not updated.**
   - Changed text: `add.go`, the "With opts.AdoptOnly, Add re-makes a live pair…" paragraph around lines 185–191.
   - Evidence: it describes the `ErrNotLive` refusals but not the new rule that the adopted warp branch is the one `PairTaskBranchOrOrigin` returns, which may carry another machine's prefix. `ErrNotLive{Branch: warpBranch}` and the weft refusal now name that branch rather than `BranchPrefix+slug`.

3. **LOW — `doc.go:763–765` and the `PairTaskBranchOrOrigin` doc comment are stale.**
   - Changed text: `doc.go` says an unnamed pair "is named by the first of its names origin holds", and "A pair's names are the current `branch_prefix` plus slug and the bare slug". The `PairTaskBranchOrOrigin` doc in `pairbranch.go` says the same.
   - Evidence: the origin lookup now also takes the single origin branch `<other prefix>/<slug>` whose records branch the weft origin holds, and errors on more than one. The `doc.go` machine-key sentence does not mention it either.
   - The `TaskBranchOnOrigin` doc in `pairparts.go:75` has the same gap: "current branch prefix, or the bare slug". Its behaviour changed because `originTaskBranch` now falls back to the other-prefix lookup.

4. **LOW — The `createRefusal` call in `battencli/wire.go:1170` uses `cfg.BranchPrefix+slug`.**
   - Changed text: not changed by the fix.
   - Evidence: this is a create path, which forks a new pair, so it is plausibly correct. I did not check `createRefusal`'s use of the name, and the call is listed only for completeness.

Calls made:
1. Read `add.go` 170–200, `doc.go` 758–770 and `remoteleftover.go` 75–95: my context lacked the `Add` doc comment, the `doc.go` prose and `RecordsBranchLive`.
2. Read `wire.go` 1160–1175, `remove.go` 600–612, `pairparts.go` 50–56 and `taskbranchdiscard.go` 66–76: my context lacked the other callers of the changed lookups.
3. Ran `git diff HEAD~1 --stat` and grepped for `RecordsBranchLive(` callers: my context lacked the commit's file set and the callers.
