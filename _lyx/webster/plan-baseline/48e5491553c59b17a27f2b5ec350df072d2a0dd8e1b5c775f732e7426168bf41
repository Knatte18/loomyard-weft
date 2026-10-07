# Card 10 — round-gate

**Edit:**
- `internal/loomshed/verifygate.go`
- `internal/loomshed/verifygate_integration_test.go`
- `internal/loomshed/seam_enforcement_test.go`
- `internal/loomshed/doc.go`
- `internal/shedrecipe/entries_gate.go`
- `contracts/recipes/loom-recipe.yaml`
- `pattern/PATTERN-verified-tree.md`
- `pattern/PATTERN-test-speed.md`
- `pattern/PATTERN-gate-self-check-parity.md`
- `PATTERN.md`

**Uses:**
- `plan:internal/verifytree#LatestPass`
- `plan:internal/commentlint#Lint`
- `plan:internal/impactset#Derive`

**Intent:**
Decisions D1, D2 and D7 at the Webster-Burler round gate: the gate `NewVerifyGate` builds runs the comment lint, then the impacted-set command, instead of the plan's `## verify:`.
Webster's gate after the last batch, Publish and Finalize keep the full plan verify.

At each arrival the gate:

1. parses the plan for its verify command, passing with the logged warning as today when there is none;
2. reads the base from `verifytree.LatestPass` for the plan's verify command;
3. calls `impactset.Derive` over the worktree and that base; a misplaced `//lyx:guard` fails the gate with the error's file and line as its findings;
4. when the derivation names a usable base, runs `commentlint.Lint` from it to HEAD and fails the gate on any finding, listing each file, line and text, before any test runs; with no usable base the lint passes, as D7 states;
5. runs the derived command, or the plan's verify command on a fallback (logging the reason), through `verifytree.Verify` with the site label it is told (`Webster-Burler gate`) and `BaseCommand` set to the plan's verify command, so the round's pass never evicts the plan-verify entry it diffs from.

`StatusDirty` and `StatusFailed` map to findings as today, so a dirty tree is still refused.
Bound, since the gate now runs less than the full verify: the narrowing is `impactset`'s, every fallback it lists runs the full verify, and the landing gates still run the full verify on the tree that lands.
The round only compiles the `tmux` and `llm` tiers and never runs them: a failure only a `tmux` test shows is caught where `publish_verify` runs the tier, at Publish, and a failure only an `llm` test shows passes every gate and is left to the by-hand run.

`seam_enforcement_test.go`'s allowlist admits `internal/impactset` and `internal/commentlint`.
`entries_gate.go`'s doc for the "verify" gate and the recipe's comment on Webster-Burler's `gates:` say what the gate now runs.
`pattern/PATTERN-verified-tree.md` states that the Webster-Burler gate runs the derived round command through `Verify`;
`PATTERN.md`'s `PATTERN-verified-tree` line says "a verify command" rather than "a plan's verify command".
`pattern/PATTERN-gate-self-check-parity.md`'s Pairs list adds the round gate to the lint pair card 7 enters, and its Webster-Burler `verify` gate line says the gate runs the lint and the derived command through `verifytree.Verify`.
`pattern/PATTERN-test-speed.md`'s tier table's "runs under" column follows D1 (`tmux`: Publish's `publish_verify` and by hand; `llm`: compiled by every gate, run by hand), and the background gains D1's gate-per-tier table.

Tests in `verifygate_integration_test.go`, over a hub fixture whose worktree holds a small Go module:
a round touching one package since the recorded plan-verify pass runs the narrowed command (read from the verify log or the record's command), not the plan's;
with no recorded pass it runs the plan's command and the lint passes;
a wrapped new comment fails the gate before the tests run;
a round pass leaves the plan-verify entry in place and a second arrival on the same tree skips;
a dirty tree is refused;
a misplaced marker fails with its file and line.

**ImpactSummary:** Changes what the one recipe row with the "verify" gate runs per round; Webster's gate, Publish and Finalize are untouched.
