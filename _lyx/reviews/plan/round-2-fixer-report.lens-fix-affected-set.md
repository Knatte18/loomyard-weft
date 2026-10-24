```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. I reran each set finding's search and checked that the fix covers every member.

Unchecked findings: none. F1 (cards 3, 4, 5, 14, 20) and F2 were rerun. F3 and F4 have no set premise.

**F1 — covered:**
- **Card 3:** the fixer's own search for `GateCall(` and the old `gate test --wait --tier` literal already covered the module. It added `loom-status-spec.md` beyond the review. The remaining hits are placeholder inputs, not rendered-call readers: the `template_test.go` constants, the `verifygate_test.go` fakes and the landingshed fixed commands.
- **Card 4:** the only other seam lists, loomcli's `intentionallyNil` and `envkit_test`'s `want`, hold no `Round` entry. So battencli's list is the only member to cover.
- **Card 14:** every caller of the re-signed `FindGuardTests`, `ModuleGuardTests`, `guardNamesInFile` and `targetsFor` is on card 14 or now listed. That includes `TestModuleGuardTests_OverAModule`, whose `{g: [TestGuard]}` comparison lives in `guardscan_integration_test.go:35`.
- **Cards 5 and 20:** covered by the fixer's own searches, as already shown in context.

**F2 — one gap:**
- LOW — the changed text is card 31's follow-up entry text, which now ends "...and the `tmux` and `llm` terms of `internal/testkit/tmuxkit/allow_tmux.go`'s constraint."
  - Summary: the rerun search finds old-spelling consumers outside Go code that also break once the follow-up drops the old tags, and the follow-up list omits them.
  - Evidence: `crucible/README.md:132-144`, `crucible/orchestrator-prompt.md:119,122` and `crucible/review-prompt-template.md:135,149-150` tell a session to run `go test -tags tmux` or `-tags llm`. Once those tests are tagged `tier2` and `tier3`, that command compiles none of them and runs nothing.
  - Not counted: `docs/benchmarks/*` hits are measurement reports, a prose exception. `contracts/specs/loom-plan-spec.md:343-344` is already on card 22.

Tool calls:
1. `git grep` for `-tags` holding an old spelling outside Go files, plus `--tier` in docs and specs: context held only Go-file searches.
2. Corrected pathspec rerun of call 1: the first form errored.
3. `git grep` for callers of card 14's re-signed functions: context held only the `.Guards` search.
4. Function list of `guardscan_integration_test.go`, plus every `intentionallyNil` and `NilSeams` list: to name the enclosing test and find sibling seam lists.
5. `Round` entries in the loomcli and envkit seam lists: to confirm no other list names the deleted field.
