```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the affected-set searches for F1 and F2 and check the fix covers every member.

**Findings**

- **MEDIUM** — `internal/fabricengine/addparent.go:53` and `internal/fabricengine/originreads.go:41,100`.
  - Summary: three other readers of `pairBranchNames(cfg, slug)` still match only the current prefix and the bare slug, so they miss a pair made under another machine's prefix.
  - Evidence: the grep shows these three call sites beside the ones the fix reached. The fix changed `PairTaskBranchOrOrigin`, `originTaskBranch`, `originPairBranch` and `Add` only.
  - Not checked: I did not open them, so whether each is a slug-to-branch lookup that needs the fallback is unconfirmed.
  - Why it matters: F1's premise is every slug-to-branch lookup. A parent given as `a/<parent>` on another machine, or an origin read in `originreads.go`, would still fail there.

- **LOW** — `internal/fabricengine/remoteleftover.go:84`.
  - Summary: `weftBranch := RecordsBranchName(t.cfg.BranchPrefix + slug)` still builds the branch from the current prefix alone.
  - Evidence: grep match for `BranchPrefix +`.
  - Not checked: I did not open it, so I cannot say whether it can run on a pair made under another prefix.

- **LOW** — `internal/battencli/wire.go:1170`.
  - Summary: `createRefusal(err, slug, cfg.BranchPrefix+slug, …)` uses the current prefix.
  - Evidence: grep match.
  - Not checked: it is for creating a new pair, where the current prefix is correct. I did not open it, so I am not certain it needs no change.

- **F2 set: no finding.** Every caller of `slugOfBranch` goes through the function, so the one change covers them all: `pairparts.go:78`, `cleanup.go:265` and `cleanupremotewarp.go:164,215`.

**Calls made**
1. `grep -rn "BranchPrefix *+\|pairBranchNames(" internal --include=*.go`, excluding `_test.go`. It reran F1's set search for call sites that build or match a warp branch from the current prefix. My context held only the review-named sites.
