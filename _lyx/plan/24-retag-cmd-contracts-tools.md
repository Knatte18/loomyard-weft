# Card 24 — retag-cmd-contracts-tools

**Custom:**
- `cmd/lyx#`
- `cmd/testtiming#`
- `contracts/specs#`
- `contracts/stencils#`
- `tools/codestats#`
- `tools/deploy#`
- `tools/forkaudit#`
- `tools/godocreflow#`
- `tools/internal/devbin#`
- `tools/mdreflow#`
- `tools/plandag#`
- `tools/sandbox#`
- `tools/tokencount#`
- `tools/wordswap#`

**Intent:**
Retag every test file of these packages to its one tier tag (D5), the first retagging card, under the overview's `retag-rules` decision.

Each `_test.go` file holding a `Test`, `Benchmark`, `Fuzz` or `Example` function takes exactly one tier tag in its `//go:build` line: an untagged file becomes `tier0`, `integration` becomes `tier1`, `tmux` becomes `tier2` and `llm` becomes `tier3`, a platform term staying beside it (`tmux && !windows` becomes `tier2 && !windows`).
A file holding a `//lyx:guard` test takes `tierN || integration` instead (D13), so the installed `lyx`'s guard scan accepts it.
A file holding only support, such as `TestMain` and helpers, keeps its constraint; a support file two tiers share keeps a disjunction of their tags, `tier2 || tier3` for `tmux || llm`; a file whose helpers another tier's tests use is split, the helpers moving to a support file both tiers compile.
Testdata files and fixture source inside string literals are not test files and are left alone.
Every card from this one on tags each test file it adds with its `tierN` tag.

This card's Verify line checks the hub's tier commands before the card lands, since the installed `lyx` has no D7 check (D13): it fails unless the `go test` step of each tier's command in `lyx config gate --resolved` names its own `tierN` tag in its `-tags` value, fails when any tier still carries a `covers` entry, and fails when it reads no tier at all, so an empty or refused `--resolved` output never passes.
When it fails, the way forward is the Constraints' hub change, applied by the orch from the prime with `lyx config gate --set`, and then a rerun of this card.

**Verify:** lyx config gate --resolved | awk '/^    tier[0-9]+:/ { tier = $1; sub(/:$/, "", tier); seen = 1 } /^        command:/ && $0 !~ ("go test[^&;|]*-tags[ =][^ ]*" tier "([,\" ]|$)") { bad = 1; print "the go test step of " tier " lacks -tags " tier } /covers:/ { bad = 1; print "a tier still carries covers" } END { if (!seen) { print "no tier read from lyx config gate --resolved"; exit 1 } exit bad }'
