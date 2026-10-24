# Card 22 — tier-plan-docs

**Prosa:**
- `internal/gateslot/doc.go`
- `pattern/PATTERN-test-speed.md`
- `docs/overview.md`
- `contracts/specs/loom-plan-spec.md`
- `internal/webstercli/verify.go`
- `internal/darnengine/config_test.go`

**Intent:**
State the tier plan per recipe in code docs (D12) and bring the tier and gate prose up to the gate keys.

`internal/gateslot`'s doc gains a Tier plan section holding D12's table, which key runs at which row with which default scope:
loom's Webster per card (`card`, `card`), per batch (`batch`, `card`), its verify fixers (`webster-fix`, `diff`) and its end gate (`webster`, `all`); loom's Merge-In (`merge-in`, `all`); loom's and darn's Code-Burler-Fix (`code-burler-fix`, `diff`) and Finalize (`finalize`, `all`); darn's Darn-Write (`darn-write`, `diff`); and darn's Darn-Write, Code-Burler-Fix and Finalize over a diff wholly under `tree_scope`, which run their row's key over the touched trees' targets.
It says darn and loom share the keys of the rows they share, that Webster's end gate runs whole while darn's writer runs the diff by operator decision, that a row's `gate_key` overrides its row name, and that the merge fixer runs its row's key.
Cards 1, 3 and 14 already brought its Config section up to the keys, the rendering and the retired `guard_tier`, so this card adds the tier plan only.
The file comment of `internal/webstercli/verify.go` names the `webster` gate key where it names the `final` gate, and the file comment of `internal/darnengine/config_test.go` names `TreeScopedTargets` where it names the deleted `TreeScopedCommand`.

`PATTERN-test-speed.md`'s tier table names each tier's tag, `tier0` through `tier3`, says tags are disjoint so each test runs in one tier, gives the one-tag rule and the `tierN || integration` transition, and replaces the gate table with the gate keys.
`docs/overview.md` names the gate keys where it names the `landing` gate and the round gate.
`loom-plan-spec.md`'s verify model section describes the keys in place of the `card`, `batch`, `round`, `final` and `landing` lists, and a `**Verify:**` line that tests as one `lyx gate test --wait` call with a gate key or a tier list and a scope form.
