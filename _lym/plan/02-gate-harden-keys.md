# Card 2 — gate-harden-keys

**Create:**
- `plan:internal/gateslot#HubSetsGate` -> `func HubSetsGate(dirs configengine.Dirs, key string) (bool, error)`
- `plan:internal/gateslot#TestHubSetsGate` -> `func TestHubSetsGate(t *testing.T)`

**Edit:**
- `internal/gateslot#goReadGates`
- `internal/gateslot#templateDefaults`
- `internal/gateslot#TestConfig_TierUse`
- `internal/gateslot/doc.go`
- `pattern/PATTERN-test-speed.md`

**Extends:**
- `internal/gateslot/template.yaml`

**Uses:**
- `internal/testkit/permkit#Chmod`

**Intent:**
Ship the three Hardener gate keys in the gate template, the way `code-burler-fix` is (D3, D4).
`template.yaml`'s `gates:` gains `harden-live: {tiers: [0], scope: diff}`, `harden-live-copies: {tiers: [0], scope: diff}` and `harden-burler-fix: {tiers: [0], scope: diff, reports: [reach, redundancy, mutation]}`, and the comment listing the keys lym reads names them.
`harden-burler-fix` carries `code-burler-fix`'s reports, since it is the same fix gate over a round diff.
The two live keys carry no `reports`: their calls run the `packages` targets after the `verify` gate has already run the reports over the round diff.
A key may name only a declared tier, so the template names tier 0; a hub sets the live keys to its live tier in its own `gate.yaml`.

`internal/gateslot/doc.go`'s sentences on the template's keys are rewritten to match:
"The template gives `webster-fix`, `darn-write`, `code-burler-fix`, `harden-burler-fix`, `harden-live`, `harden-live-copies` and `pr-review` the `diff` scope.";
the `reach` sentence names `harden-burler-fix` after `code-burler-fix`;
and the `mutation` sentence becomes "It lists `mutation` last on `webster-fix`, `darn-write`, `code-burler-fix`, `harden-burler-fix` and `pr-review`, every key of scope `diff` but the two live keys, which carry no reports.".
Its site table gains two rows: `hardener`, `Harden-Burler-Fix, verify gate`, `harden-burler-fix`, `diff`;
and `hardener`, `Harden-Burler-Fix, live gate`, `harden-live, harden-live-copies`, `diff`.
`pattern/PATTERN-test-speed.md`'s gate-key table gains the same two rows:
`harden-burler-fix` for Harden-Burler-Fix's `verify` gate, and `harden-live`, `harden-live-copies` for its `live` gate's merge-bar run and concurrent copies, each with template scope `diff`.
`goReadGates` lists the three keys after `code-burler-fix`, so `TiersValue` renders them.
`templateDefaults` in `config_test.go` gains the three entries, so `TestLoadConfig`'s template row matches, and `TestConfig_TierUse`'s `TiersValue` want text gains the three keys wherever its config carries them.

`HubSetsGate` reports whether the hub's own `gate.yaml`, the hub base layer alone, sets `gates.<key>`, read through `configengine`'s layer stack for the gate spec; a template default, a worktree layer and a machine layer never count, so only the hub's own file passes Preflight's hub-key check (D2).
Card 5's Preflight check calls it to refuse a hub that left a live key at the template's tier 0.
`TestHubSetsGate` is tier0: a hub file without the key answers false, one setting it answers true, a worktree layer setting it over a hub file without it answers false, and a hub file made unreadable through `permkit.Chmod` returns its error.

**ImpactSummary:** Three template keys and one read-only lookup in gateslot; existing keys and their readers are unchanged.
