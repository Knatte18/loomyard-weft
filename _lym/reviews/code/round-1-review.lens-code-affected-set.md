```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, meaning what the diff forces outside the lines it touches.

**Per change, the set found**

1. `finishWrite` signature, now taking a `render` argument.
   - Callers: `setCmd`, `setOverride`, `removeCmd` (both branches). All were updated.
   - Covered.

2. `write` and `syncWarning` removed, replaced by `saveLocked` and `syncAfterSave`.
   - Searched the whole tree for `syncWarning`: no references remain.
   - Callers of the old `write`: `ChangeStore` and `changeStore` only. `changeStore` now keeps the sync warning and does not sync on a refused change.
   - `internal/orchcli/rules.go:63` is the only external `ChangeStore` caller, and `TestHeddleCLI/ChangeStoreReturnsErrorsAndWarning` asserts its warning.
   - Covered.

3. New `set` and `remove` output keys: `rendered`, `failed`, `render_warnings`.
   - No code or stencil outside `internal/heddlecli` reads `lym heddle set` or `remove` output.
   - `orchcli` reads only `ChangeStore`'s warning string.
   - Covered.

4. New `--no-render` flag.
   - Wired into the help text of `setCmd` and `removeCmd`, and into `doc.go`.
   - Nothing outside the package lists these flags.
   - Covered.

5. `renderInto` extracted from the `render` verb.
   - The `render` verb's check for "lym heddle render" in the message still works.
   - `renderInto` returns `seed the exclude lines: %w` without a worktree name, and the verb's wrapper adds `heddle: render <name>:`.
   - Covered.

6. Docs the diff updated.
   - `doc.go`, the group `Long` text in `cli.go`, `heddleengine/seam.go`, and `PATTERN-heddle-instructions.md` lines 23–24 all describe the new fan-out.
   - The `PATTERN` entry and its `Rule:` summary line do not conflict. The declared-changes list is `none`.
   - Covered.

7. Docs elsewhere that describe the old behaviour (searched `docs/overview.md`, the glossary, `contracts/orchrole/orch-role.md`, `PATTERN-lym-files.md`, `PATTERN-config-layers.md`).
   - None describe the old staleness or hand-render behaviour of `set` and `remove`.
   - Nothing was left wrong.

8. `internal/hubreconcile/walk.go:299` says "the next `lym heddle set` or `lym heddle remove` renders it".
   - It is still accurate for the render the hub reconcile case needs.
   - With `--no-render` or a failed worktree the claim is weaker, but it was already conditional on the "next" write.
   - Not a finding.

9. `orch rules`, which writes through `ChangeStore` and so renders no worktree.
   - It writes the `orch-prompt` target, which is not a file target, so no rendered file goes stale.
   - `doc.go` and `ChangeStore`'s comment both state "renders no worktree".
   - Covered.

**Findings**

1. **LOW, `scope`.** `finishWrite` renders into every listed non-prunable entry using `filepath.Base(entry.Path)` as the worktree name. Evidence: `internal/heddlecli/verbs.go:458-466`.
   - Other hub-wide walks guard against more than prunable entries. `fabricengine.CodeWorktrees` (`internal/fabricengine/codeworktrees.go:33-45`) also skips entries whose directory is gone. `orchcli/live.go:32` also skips worktrees in staging via `InWorktreeStaging`.
   - A `.fabric-wt-staging-*` worktree is listed while a pair is being made. The fan-out would call `lymcwd.NamedWorktree` and `EventFor` on it, which seeds exclude lines and renders, and it would report a `failed` entry.
   - The doc says "live worktree", so this is a mismatch with the set of worktrees other code treats as live.
   - Fix direction: skip `fabricengine.InWorktreeStaging(entry.Path)` entries, and ideally reuse the existing live-worktree filter.
   - Whether staging entries actually appear in `List` I took from the `orchcli/live.go:30-33` comment, not from running it.

No other member of the affected set was left wrong. There is no stencil change, and no `Pattern-Departure:` trailer is needed, since the commit message has none and nothing in the diff departs from a pattern.

**Tool calls**
1. `grep` over md, go, yaml and json for "hand render", "lym heddle render" and similar wording. My context did not hold other readers of these strings.
2. `grep` for the `heddle` doc files, plus a read of `orchcli/rules.go` and `orchcli/doc.go`. My context lacked the `ChangeStore` caller's doc.
3. `grep` of `heddle` in `orch-role.md`, `overview.md`, the glossary and two other `PATTERN` files. This checked for docs describing the old `set` and `remove` behaviour.
4. `sed` of `overview.md:344` and `:683`, and the glossary `:49`, to confirm they do not describe the old behaviour.
