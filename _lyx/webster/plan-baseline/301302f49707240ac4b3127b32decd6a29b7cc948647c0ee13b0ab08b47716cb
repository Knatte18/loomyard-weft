# Card 1 — verifytree-per-command

**Edit:**
- `internal/verifytree/verify.go`
- `internal/verifytree/doc.go`
- `internal/verifytree/verify_integration_test.go`
- `pattern/PATTERN-verified-tree.md`

**Create:**
- `internal/verifytree#LatestPass`

**Intent:**
Decision D3: the verified-tree record holds one entry per command instead of one entry overall.
Each entry names the command, HEAD's tree, HEAD's commit and the time of its pass.
The skip rule is unchanged: `Verify` skips when an entry for the same command names HEAD's tree.

Retention on a write: the pass replaces the entry of its own command, and drops every entry of another command that names a different tree, except the entry of the base command the caller names.
The caller names the base command on `Site` as a new field, `BaseCommand`; empty names none, which is what today's plan-verify sites pass, so no existing caller changes.
Without the rule each round's impacted-set command, which differs per round, would add an entry no later skip can match.

`LatestPass` returns the entry of `command` (its tree, commit and pass time as a new `Pass` struct), false when the record holds none.
The round gate (card 10) reads its diff base from it.
A record in the old single-entry format, or a malformed one, reads as no record, which only costs one full run.
`Verify` resolves HEAD's commit beside HEAD's tree before running, and records the pass only when both still match after the run, as it does for the tree today.

Tests, in `verify_integration_test.go` since `Verify` spawns git: a round-command pass leaves the plan-verify entry in place, and a second call on the same tree and command skips;
a write drops another command's entry naming a different tree, keeps the `BaseCommand` entry, and replaces its own command's entry;
`LatestPass` returns the commit of the pass; an old-format record reads as none.

`pattern/PATTERN-verified-tree.md` states that records are per command, each with its commit.

**ImpactSummary:** Changes the on-disk record format and adds one `Site` field; every existing `Verify` caller compiles unchanged and keeps its skip behavior.
