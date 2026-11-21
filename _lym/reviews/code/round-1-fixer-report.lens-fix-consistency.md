```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set against its siblings for consistency (lens fix-consistency).

Findings:

- LOW — `internal/raddleparser/check.go:98`, the `BoardContext` doc: "`ValidateSlug` returns the board's refusal of a slug a filing would make."
  - F3 made the `raddlecontext.go` doc and the spec say the pair rule is applied by `lym board` writes and pair creation, not by Raddle-File's own board write.
  - This twin doc was left as it was. It still presents the check as the board's own refusal at filing.
  - The wording is not contradicted outright, since the filed entry's pair would be refused later, but it is the one place F3 did not reach.
  - Evidence: `check.go:98-99` against the reworded `raddlecontext.go:33-34`.

Everything else agrees:

- The spec's rule sentence at line 65 and the `slug-invalid` bullet at line 110 now state the same rule.
- The `raddlecontext.go` comment's claim that `lym board` writes apply the pair rule holds. `boardcli/cli.go:164` sets `cfg.ValidateSlug` with a fabric-config load.
- The keep marker's reason matches the reason the review proposed. It sits below a `//` blank line, so it is directly above the `func` line as the PATTERN requires.
- The new `docs` row relies on `SeedFabricConfig` writing `pathspec: docs`. The tier-1 gate ran `TestRaddleBoardContext` and passed.

Calls made:
1. `grep` for `slug-invalid` and the old board-write wording across Go and markdown files, plus `git diff --stat`, to find twin statements of the rule that the context lacked.
2. `grep` for `ValidateSlug` in `raddleparser` and a read of `boardcli/cli.go:158-168`, to check the `BoardContext` doc and verify the `lym board` claim.
