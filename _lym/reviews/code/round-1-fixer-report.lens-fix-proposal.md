```yaml
lens: fix-proposal
outcome: reported
```

No findings.

I tried to break both proposals and could not.

**F1.** The added sentences in `internal/orchcli/rules.go` and in the "Orch rules" section of `orch-template-role.md` add no `%` verb, so the `fmt.Sprintf` in the long help is unaffected.
- The role-stencil tests I found check fixed strings in the stencil, such as `shuttleengine.MessageTail`; the added sentence does not touch them.
- Nothing else in `internal/orchengine/prompt.go` or `orch-template-start.md` still names `_lym/orch/rules.md`, so no sibling text contradicts the new sentence.
- The fixer did not run `./contracts/stencils` tests after editing the stencil. I judged this unlikely to matter, but I did not run them.

**F2.** Splitting `ChangeStore` into the `heddleCLI.changeStore` method leaves its behaviour unchanged, because the public function builds the same zero-seam receiver.
- The new tier1 subtest shares one hub across the outside-a-hub, refused-change and failed-sync-start cases. They do not interfere: the first two save nothing, and only the last writes.
- The `CwdOutsideAHub` row in `internal/orchcli/rules_test.go` relies on `t.TempDir()` not being inside a hub, which holds.

Further tool calls (2 of 6):
1. `grep` for `orch-template-role` and `_lym/orch/rules.md` across the worktree: checked whether anything else pins or names the stencil or the old rules file; my context lacked that.
2. `grep` in `contracts/stencils/govalues_test.go`, `markers_test.go`, `orch-template-start.md` and `internal/orchengine/prompt.go` for `_lym/orch/rules.md`, then `govalues_test.go` for `orch-template-role`: checked what the stencil tests assert; my context lacked that.
