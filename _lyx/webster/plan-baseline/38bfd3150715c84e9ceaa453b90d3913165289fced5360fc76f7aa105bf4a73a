# Card 8 — diff-base-chain

**Edit:**
- `internal/gatecli#computeScope`
- `internal/gatecli#scope`
- `internal/gatecli#planTierRuns`
- `internal/gatecli#runTest`
- `internal/gatecli#TestGateTest_Scopes`
- `internal/gatecli/doc.go`

**Create:**
- `internal/gatecli#greenRecord.newestWholeAncestor`

**Uses:**
- `internal/gatecli#greenRecord`

**Intent:**
A `diff` scope takes its base per listed tier from the green record (D2).

For each listed tier, the base is the newest commit that is an ancestor of HEAD and whose tree the record holds whole-green for that tier under its current fingerprint, with empty args and the same module directory, found by `greenRecord.newestWholeAncestor` with ancestry read through `gitrepo`'s `IsAncestor`.
Without one, the base is the merge base with the parent branch, today's computed form; without a parent, the tier runs whole.
A narrowed green never becomes a base, and neither does a whole entry recorded under `-- -run X` or under a since-changed tier command, so the chain stays sound.
Tiers that land on the same base share one `impactset.DeriveTargets` derivation: `scope` carries per-tier modules, keyed by base, and `planTierRuns` takes each tier's share.
Each derivation prints one stderr line, `gate test: diff scope from <base> for <tiers>`, naming the base and the tiers that share it, so a caller sees which base each tier diffed from.
The `card` and `all` scopes are unchanged; a dirty worktree reads no record and so takes the merge-base fallback.

`TestGateTest_Scopes` gains rows: a `diff` run after a whole-green commit diffs from that commit; with none it diffs from the parent merge base; without a parent it runs whole; a whole entry under `-- -run X`, and one under a since-changed tier command, are never taken as a base; a whole entry recorded in another module directory is never taken; a dirty worktree diffs from the parent merge base though the record holds a whole-green ancestor; and two tiers with the same base print one derivation line naming both, while two tiers with different bases print two.
The gatecli doc's Scope section states the chain.

**ImpactSummary:** Changes how the `diff` scope picks its base, per tier and shared where equal; only `--gate` calls with a `diff` key and the computed form read it.
