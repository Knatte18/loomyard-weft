```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 and F2 change set against its surrounding docs and twins (lens fix-consistency).

**no findings**

- The F1 sentence says the same thing in both places. The `rules` command's long help in `internal/orchcli/rules.go:38-40` and the "Orch rules" section of `contracts/stencils/orch/orch-template-role.md:272-273` both say a `_lym/orch/rules.md` file, "hub-wide on Bolt or per orch", is no longer read. Both name the same carry-over route: copy to a draft, then `lym orch rules --from`.
- The new sentences don't contradict the nearby text. Role line 239 says a draft starts as a copy of `lym heddle get orch-rules`. The migration sentence covers a hub with no such entry.
- No sibling still describes the old two-file scheme. A grep for `orch/rules`, `orch rules` and `orch-rules` over `.go`, `.md` and `.yaml` finds none in `internal/orchcli/doc.go`, `internal/orchengine/doc.go`, `internal/heddlecli/doc.go` or `pattern/PATTERN-heddle-instructions.md`. These all describe the heddle entry.
- The only stale references are in `docs/research/` and its bench prompts. The review's own finding says they are measurement records, which the prose rules exempt, and the fix didn't touch them.
- The `internal/heddlecli/cli.go` change (`ChangeStore` now delegates to `changeStore`) keeps the doc comment accurate. `internal/heddlecli/doc.go:14` still describes `ChangeStore` correctly.
- The `rules_test.go` header was updated to match its new `CwdOutsideAHub` row. The `heddlecli` tier1 file header still covers a "Bolt write lock and sync around a write", so the new subtest fits it.

Calls:
1. `grep` for `orch/rules|orch rules|orch-rules` across the repo, to find twins and stale references outside the files already in my context.
