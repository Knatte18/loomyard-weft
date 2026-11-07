```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change set for consistency with its siblings, meaning other template copies in `TestWorktreeSeam_Events` still taken after the seam install.

**Finding 1: MEDIUM.** The hoist left `hubforge.CopyHub` copies after the install.
- **Changed text:** the hoisted block before `seam := &fakeWorktreeSeam{t: t}` in `worktreeseam_integration_test.go`. It covers only `newOriginHub` and `newMovesPair`.
- **Twin left behind:** the subtests "A pull over rewritten warp history…" (`:363`) and "A hub config that cannot be read…" (`:445`) call `buildReconcileFixture` after the install. That helper calls `hubforge.CopyHub(t, hubforge.Shape{Anchor: "."})` at `pull_integration_test.go:39`.
- **Why it matters:** the F1 premise is that the first copy of a template builds it under whatever seam is installed. `hubforge/doc.go:28` says "The first CopyHub, CopyRegistered or SharedHub of a key builds its template once per test binary." If the `Shape{Anchor: "."}` template has not been built earlier in the binary, as under `-run TestWorktreeSeam_Events`, its build runs under the fake seam. A hub build goes through `cloneHub` and `CloneAndWire`, which fire the `Made` event.
- **Effect on assertions:** those subtests filter by `weftFixture.Path`, so their assertions are unaffected. The criterion the review cites, "each test that installs a process-global seam takes its copies before installing it", is still not met for this copy.
- **Not verified:** I did not read `CopyHub`'s build path, so I don't know whether a Shape template's first build really reaches the seam.
- **Not a twin:** `hubforge.NewHub` and `CloneSecond` at `:187-188` run after the install on purpose. The clone's `Made` event is what the first subtest asserts. Both call `cloneHub` directly and take no template copy.
- **Possible fix, if confirmed:** take one `hubforge.CopyHub(t, hubforge.Shape{Anchor: "."})` before the install, to warm the template. Alternatively, record that this copy is exempt.

No other findings.
- `boardkit.InstallReplay(t)` moved to the top of the test and now covers the first subtest too. That is harmless. It matches `TestReconcileOrigin_Moves`, which installs replay before its copies (`originreconcilemoves_integration_test.go:151`).
- The new local names don't collide with the existing `hub`, `fab`, `second` or `seam`, and the subtests' `origin` and `pair` shadows stay local.
- The file header and the test's doc comment (`:164-166`) still hold.

Tool calls (2 of 6):
1. `grep` of `hubforge` `NewHub`, `CloneSecond` and `CopyRegistered`, and of `buildReconcileFixture`'s copy calls. My context lacked those helpers' bodies.
2. `sed` of `hubforge/hub.go:245-300` and `pull_integration_test.go:32-60`. My context lacked how `NewHub` and `CloneSecond` copy, and which copy call `buildReconcileFixture` makes.
