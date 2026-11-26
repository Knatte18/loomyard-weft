```yaml
lens: code-tests
outcome: reported
```

The code-tests lens (one test row per behaviour the diff adds or changes) found one gap and one prose assertion.

**Per behaviour, the test that pins it:**
1. **Refusal names the kept-cd cause and the way forward, for a cwd that is a subdirectory under anchor ".":** pinned by `TestCheckCwdAnchorGate`, row "subdirectory of an unanchored root errors" (`internal/lymcwd/gate_test.go:53-63`, checked at :98). It compares the whole message, so it fails on the merge base's text.
2. **The way forward's `cd` target is the anchored directory (`anchorAbs`), not the worktree root, when the anchor is a subpath:** no test. Only the row with anchorRel "." sets `wantMessage`, and there `anchorAbs` equals `worktreePath`. The rows "sibling directory of a subpath anchor errors" (:71-75) and "worktree root above a subpath anchor errors" (:77-81) check only `errors.Is`.
3. **The refusal still wraps `ErrCwdOutsideAnchor` and keeps its opening text:** pinned by the `errors.Is` check at `gate_test.go:89` and by the message prefix in the row at :58.
4. **The message reaches creel and other verbs verbatim through `preflight.ResolveMode`:** the diff does not change this path. `internal/preflight/preflight_integration_test.go:653` covers it through `errors.Is`. No new behaviour, so no gap.
5. **The shipped entry `lym-shell-directory` is in the shipped layer, at order 30 after `lym-plan-edits-before-gate`, aimed at `agents-file` alone:** pinned by `TestStore/ShippedEntriesArePinned`, row at `internal/heddleengine/store_test.go:188`.
6. **The shipped entry's body text (its claims and the absolute path, `git -C` and `env -C` advice):** no test. That is right, because static prose is exempt.

**Findings:**

- **LOW · design · `internal/lymcwd/gate_test.go:71-81`.** No row pins the way forward for a subpath anchor. A mutant that put `worktreePath` instead of `anchorAbs` after `way forward: cd` (`internal/lymcwd/anchor.go:117-118`) passes every row. The only full-text row has anchorRel ".", where the two paths are equal.
  - Fix: give the "worktree root above a subpath anchor errors" row a `wantMessage` whose way forward reads `cd home/user/repo/backend`.
- **LOW · consistency · `internal/heddleengine/store_test.go:188`.** The new row asserts the entry's heading text "## Keep the shell where it started", which is static prose of a shipped instruction. That is the rubric's "new assertion on the static prose" test-economy item.
  - Mitigation: the table already pins full headings for the other agents-file entries (:186-187), and a row is needed to pin the slug, order and target.
  - Fix: make the row `levelOnly` with `"## "`, as the `lym-glossary` row does (:194), so it keeps checking slug, order, target and heading level without fixing the wording.
- **NIT · consistency · `internal/lymcwd/gate_test.go:18-25`.** The doc comment of `TestCheckCwdAnchorGate` still describes only the accept/reject triples and `ResolveWorktree` being ungated. It does not mention that the test now pins the refusal's whole text for the subdirectory row.

Tool calls: none; my inherited context held every file this lens needed.
