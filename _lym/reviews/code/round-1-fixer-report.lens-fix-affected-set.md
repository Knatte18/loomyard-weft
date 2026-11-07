```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F3:** I searched `.go`, `.md` and `.yaml` for sites that still describe `{targets}` alone.
  - Every non-test site now names both placeholders: `gateslot/tiers.go:187`, `gateslot/doc.go:138`, `gateslot/template.yaml:7`, `gatecli/tiers.go:81` and `:167`, and `gatecli/doc.go:99` and `:310`.
  - The remaining `{targets}`-only wording is in three test names: `tiers_test.go:233`, `:299` and `:461`, and `test_integration_test.go:1699`. They describe commands that really hold no `{targets}`, so they are not stale.
  - `docs/dotnet-hub.md:16` and `:32` describe dotnet commands, where `{test_targets}` does not apply.
- **F2:** Other lists of test-function prefixes already include `Example`: `loomshed/docgo.go:26`, `planparser/vocabname.go:86` and `cmd/testtiming/spawnscan.go:184`. The fix makes `impactset` consistent with them.

Tool calls:
1. grep for `{targets}`-only wording across `.go`, `.md` and `.yaml`: the review's F3 site list was only a sample of the sites.
2. grep for the `"Benchmark"` and `"Fuzz"` prefix lists: to check the `Example` addition matches the other prefix sets.
