```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F1: no Go or markdown file outside `_lym` and `.lym` still uses the removed `SessionCauseLivenessUnproven`/`liveness-unproven` cause. The `LivenessUnproven` hits are the `Liveness` enum value, which is a different symbol. The only text hit is a dated research doc, `docs/research/codex-engine-trial-hooks-guards-skills-2026-11-03.md:65` and `:135`. It is a historical record, so I left it alone.
- F2: the only code that builds the `--anchor` refusal is `internal/shuttlecli/run.go`. `internal/reedcli/add.go:80` is reed's own validation, which accepts `hidden`, so it is not a twin of this change. The `run.go:202` flag help still says hidden is refused, which matches the new message. The `AnchorOverview` and `AnchorEmpty` rows in `cli_test.go` assert only `own-window`, which the plain message still contains.

Unchecked findings: none.

Tool calls:
1. Bash grep for `liveness-unproven` and `LivenessUnproven` across the repo. My context lacked the full set of sites.
2. Same Bash call, second grep for `reed starts no command` and `invalid --anchor`. My context lacked the full set of sites.
