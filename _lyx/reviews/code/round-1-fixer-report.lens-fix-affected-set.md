```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set-premise searches for F1, F3 and F4 against the fix.

**Findings**

- **LOW** — `internal/battenshed/doc.go:59`, F4's set (text naming the orch's reed session for `--window`). The sentence still reads "with "--window" in its own tmux window of the orch's reed session". The fix reworded only `docs/overview.md:628`. `grep "orch's reed session"` over `*.md` and `*.go` outside `_lyx` finds this as the only remaining match. It states the same stale session as the overview sentence the review named, and a reader of the package doc would notice.

**Checked, no findings**

- F3: the added lines of the diff against `120316dd7` contain no `umbrella` (case-insensitive). `rename_test.go` has no `umbrella` left either.
- F1: the production `ResolveParent` callers are `orchcli/notify.go:90` and `:104`, `wiring.go:390`, `landingdeps.go:471` and `battencli/land.go:255`.
  - `notify.go:90` and `:104` feed the nearest-live walk.
  - `wiring.go:390` is the parent-review addressee, now documented in `hubgeom/doc.go` as keeping the recorded parent.
  - `landingdeps.go:471` and `land.go:255` read the parent pair's worktree itself, which `ResolveParent` is meant for.
  - `decision.go:562` is the only reader of `deps.parent`, and `orchcli.NearestLiveParent` now backs it.
  - The prompt-side parent names the review mentioned are not among these callers. Their source lives outside this grep (`internal`, non-test, excluding `hubgeom/`). `hubgeom/doc.go` now says they keep the recorded parent, so I raise no finding.

**Tool calls**
1. Checked the added diff lines for `umbrella`: the diff against `120316dd7` was not in my context.
2. Searched `*.md` and `*.go` for "orch's reed session": the repo-wide F4 set was not in my context.
3. Searched for `ResolveParent` and the `parent` seam callers: the F1 set was not in my context.
4. Read `landingdeps.go:462-482` and `land.go:250-262`: I needed to see what those two callers do with the parent.
