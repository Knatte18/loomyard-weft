```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked that every sentence the fix added or rewrote holds against the code at HEAD.

1. **MEDIUM**, card 4, the new test bullet: "`TestRender_WritesAtTheAnchorAndLeavesNoUntrackedFile` sets `Root` in its request; its git-status assertion then also covers the import record."
   - **Summary:** the claim is false. The record is written outside the prime worktree, so the prime's `git status` never sees it.
   - **Evidence:** the record path is `<ScratchDir>/heddle/codex-file-imports.json`. `EventFor` sets `ScratchDir` to `filepath.Join(RecordsWorktreePath(l, l.WorktreeName), l.AnchorRel, ".lym")` (`internal/fabricengine/worktreeseam.go:66`). `RecordsWorktreePath` returns `weftname.SiblingPath(l.HubPath, slug)`, the sibling records (weft) worktree (`internal/fabricengine/weftwiring.go:35-36`). The test runs `git status` in `hub.PrimeWorktree()` (`render_integration_test.go:68`), which does not contain that path.

2. **LOW**, card 10, the changed exception: "read from the file's head as `Render` does".
   - **Summary:** `Render` does not read the head. It reads the whole file and checks the prefix.
   - **Evidence:** `existing, err := os.ReadFile(path)` and then `strings.HasPrefix(string(existing), Marker)` (`internal/heddleengine/render.go:126,130`). The behaviour, a prefix comparison with `Marker`, still matches; only the description of how `Render` reads the file is wrong.

The other claims hold:
- **Card 7:** `sessionScopeFor` logs a plugins file it cannot read and carries on with no error (`scope.go:109-121`). Its warning goes to the logger, not into a returned warnings list, but the analogy the card draws, degrade and never refuse, still holds.
- **Card 8:** no non-test file in `internal/shuttleengine` mentions `Owner`, `RunProcDir` or `HubPath`, so neither `Spec` nor `Config` carries a run owner or a `runproc` directory.
- **Card 5:** the NUL-counted `MAX_ARG_STRLEN` boundary is the reviewer's kernel fact, and the card restates it consistently.
- **Card 4:** "two production builders and two test builders" matches the four `RenderRequest` literals that `grep` found earlier.

Tool calls:
- Read `sessionScopeFor` and `Render`'s marker read, to check the card 7 and card 10 claims; my context did not hold those lines.
- Grepped how `EventFor` sets `Root` and `ScratchDir`, to check the card 4 git-status claim.
- Read `RecordsWorktreePath` and searched for `sessionScopeFor`, to find where `ScratchDir` lives and where the function sits.
- Read `scope.go:105-130` for `sessionScopeFor`'s error handling.
