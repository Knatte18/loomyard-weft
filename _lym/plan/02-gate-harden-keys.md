# Card 2 — gate-harden-keys

**Create:**
- `plan:internal/gateslot#HubSetsGate` -> `func HubSetsGate(dirs configengine.Dirs, key string) (bool, error)`
- `plan:internal/gateslot#TestHubSetsGate` -> `func TestHubSetsGate(t *testing.T)`

**Edit:**
- `internal/gateslot#goReadGates`
- `internal/gateslot#templateDefaults`
- `internal/gateslot#TestConfig_TierUse`

**Extends:**
- `internal/gateslot/template.yaml`
- `internal/gateslot/doc.go`

**Uses:**
- `internal/testkit/permkit#Chmod`

**Intent:**
Ship the three Hardener gate keys in the gate template, the way `code-burler-fix` is (D3, D4).
`template.yaml`'s `gates:` gains `harden-live: {tiers: [0], scope: diff}`, `harden-live-copies: {tiers: [0], scope: diff}` and `harden-burler-fix: {tiers: [0], scope: diff}`, and the comment listing the keys lym reads names them.
A key may name only a declared tier, so the template names tier 0; a hub sets the live keys to its live tier in its own `gate.yaml`.
`goReadGates` lists the three keys after `code-burler-fix`, so `TiersValue` renders them.
`templateDefaults` in `config_test.go` gains the three entries, so `TestLoadConfig`'s template row matches, and `TestConfig_TierUse`'s `TiersValue` want text gains the three keys wherever its config carries them.

`HubSetsGate` reports whether the hub's own `gate.yaml`, the hub base layer alone, sets `gates.<key>`, read through `configengine`'s layer stack for the gate spec; a template default, a worktree layer and a machine layer never count, so only the hub's own file passes Preflight's hub-key check (D2).
Card 5's Preflight check calls it to refuse a hub that left a live key at the template's tier 0.
`TestHubSetsGate` is tier0: a hub file without the key answers false, one setting it answers true, a worktree layer setting it over a hub file without it answers false, and a hub file made unreadable through `permkit.Chmod` returns its error.

**ImpactSummary:** Three template keys and one read-only lookup in gateslot; existing keys and their readers are unchanged.
