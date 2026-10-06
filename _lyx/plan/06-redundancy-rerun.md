# Card 6 — redundancy-rerun

**Edit:**
- `docs/benchmarks/test-redundancy.md`
- `cmd/testtiming/redundancy.go`
- `cmd/testtiming/redundancy_test.go`

**Intent:**
Regenerate the six packages' sections of the committed report with the tool cards 1–4 built, so the prune cards start from the D1 judgment.

Run `go run ./cmd/testtiming -redundancy -tags integration,tmux -pkg ./internal/fabricengine,./internal/websterengine,./internal/reedengine,./internal/shuttleengine,./internal/loomcli,./internal/shedadapters`.
The splice must leave the header and every other package's section byte-identical; check with `git diff --stat` and a read of the diff that only the six sections changed.

Check the regenerated sections against the acceptance criteria the tool can already meet: no `internal/loomcli` test listed as "failed" (card 1), and no test of an untagged file listed with the out-of-process or unclassifiable reason (card 2).
When a test of the six is still listed as "failed", find out why: a fault of the tool is fixed in `cmd/testtiming` in this card with a test row; a test that depends on another test's side effect is left to its package's card, which fixes it under D7, and is named in this card's commit message.

The two `cmd/testtiming` targets change only for such a tool fault; with none found, this card commits the report alone.

**ImpactSummary:** Rewrites six sections of a generated report, and the tool only for a fault the rerun exposes; nothing reads the report but people and the prune cards.
