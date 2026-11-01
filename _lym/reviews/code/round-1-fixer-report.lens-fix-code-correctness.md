```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: run the code the fix changed and check the tests it added.

One finding:

- **LOW** — `internal/orchcli/rules.go:86-88`, the `warning` copy into the envelope. F2's fix named "a sync that did not start (envelope carries warning)". The added tests pin `changeStore`'s returned warning in `heddlecli`, but no test drives `writeOrchRules` with a warning.
  - `if warning != "" { fields["warning"] = warning }` can be deleted and every added or existing test still passes.
  - Evidence: the `orchcli` row `CwdOutsideAHub` stops at the `wire` error, and `TestOrchIntegration_RulesStoreInHeddle` reaches only the real-sync success path.
  - A test needs a seam in `orchcli` to inject a failing `ChangeStore`. The fix added none, so that half of the finding is unresolved.

Checked and correct:

- **`changeStore` refactor:** `ChangeStore` now calls `(&amp;heddleCLI{}).changeStore(cwd, change)`, whose body is the old one. A `wire` failure returns `"", err`, a `write` failure returns `"", err`, and success returns `c.syncWarning`. The only caller, `writeOrchRules`, is unchanged.
- **`ChangeStoreReturnsErrorsAndWarning` outside a hub:** `t.TempDir()` fails `lymcwd.Resolve`, and `wire` returns the "run lym heddle from a code worktree of a hub" text the test asserts, with no save or sync. It fails if `wire` is skipped.
- **Same test, refused change:** `refusal` reaches the caller through `errors.Is`, with no save or sync. It fails if `changeStore` swallows or re-wraps with `%v`.
- **Same test, failed sync start:** the error is nil and the warning names `lym board sync`. The entry is in the store. It fails if the warning is dropped. The `Order` fix was needed because a new entry requires one, and the earlier failure came from that.
- **Shared hub across the three cases:** only the third saves, so there is no cross-contamination. The test is parallel-safe.
- **`CwdOutsideAHub` row:** `draft.md` is created, so it passes the `ReadFile` check and the refusal comes from `ChangeStore`'s `wire`. The `os` import is added.
- **F1 text:** the added help sentence has no `%` verb, so `fmt.Sprintf` still has matching arguments. The added role-stencil sentence has no `{{`, so rendering is unaffected.

Tool calls: none; my context already held every changed file and the tier0 and tier1 gate results.
