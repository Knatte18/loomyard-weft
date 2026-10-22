# Card 3 — lenses-lens-set

**Create:**
- `internal/lenses/sets/go.yaml`
- `plan:internal/configsync#burlerFixture` -> `func burlerFixture(t *testing.T, edit func(cfg *lenses.Config)) string`
- `plan:internal/lenses#lensSetFiles` -> `var lensSetFiles embed.FS`
- `plan:internal/lenses#lensSets` -> `func lensSets() map[string][]byte`

**Edit:**
- `internal/lenses#Config`
- `internal/lenses#ConfigSpec`
- `internal/lenses#TestConfigTemplate_DecodesAndIsSelfConsistent`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#fingerprintOf`
- `internal/configreg#TestFingerprint`
- `internal/configsync#TestReconcile_MovesKeysAcrossModules`
- `internal/lenses/template.yaml`
- `internal/lenses/doc.go`
- `docs/overview.md`

**Uses:**
- `plan:internal/configengine#Overlay`
- `internal/lenses#writeLensesYAML`
- `internal/lenses#loadTemplateConfig`

**Intent:**
Wire the `lens_set` selector into the lenses module and ship the `go` layer (D2, D3).

The base/`go` split is a move: `internal/lenses/sets/go.yaml` holds a `lenses:` mapping with the five lenses `error-handling`, `test-gaps`, `performance`, `api-design` and `concurrency`, each copied verbatim from `template.yaml` as it stands before this card, and nothing else.
This card does not reword the base, so under `lens_set: go` every lens still resolves to today's prose.

`template.yaml` gains the top-level scalar key `lens_set: ""` above `lenses:`, and its header comment gains a paragraph: `lens_set` names one embedded language layer under `sets/` whose lenses and fans replace or add base entries by name, `""` picks none, only the base file may hold it (the hub file in a hub, set with `lyx config lenses --layer default --set lens_set=<name>` from the hub's prime worktree), and a name this file defines still beats the layer.
`Config` gains the field `LensSet string` with tag `yaml:"lens_set"`.

`lensSetFiles` embeds `sets/*.yaml` with `//go:embed`, and `lensSets` returns each file's bytes keyed by its base name without `.yaml`, panicking on a read error of the embedded tree, which cannot fail for an embedded path.
`ConfigSpec` sets `Overlay: configengine.Overlay{Selector: "lens_set", Documents: lensSets()}` and classifies `lens_set` `CategoryGuard`, and its doc comment says why: the layer picks the prose every review fork is given.
The guard's bound: a Guard key, so no local override and no run setting holds it, and configengine reads it from the base layer only.
`internal/lenses` gains no non-stdlib import (`embed` is stdlib), so `TestLeafInvariant_AllowlistOnly` holds.

`fingerprintOf` also hashes, for a module whose `Spec` is non-nil, its `Overlay.Selector` and each overlay document in sorted name order, name then bytes, so a binary that changes only a layer's prose changes the build stamp and `hubreconcile.Ensure` reconciles again, as for a template edit (`PATTERN-binary-change-reconcile`).
A module with a nil `Spec` or an empty overlay hashes as today's modules do.
`TestFingerprint` gains rows: a module given a `Spec` whose overlay holds one document differs from the base list; the same module with that document's bytes changed differs from it again.

`burlerFixture` decodes the template into `lenses.Config` like `lensesFixture`, applies `edit`, and marshals a document holding only the config's `lenses` and `fans`, so a `burler.yaml` built from it holds only the two keys the burler-to-lenses moves take out and the move leaves it empty, never holding `lens_set`.
`TestReconcile_MovesKeysAcrossModules` builds `burlerToday` and `burlerOldTemplate` with it; `lensesFixture` keeps marshalling the whole `Config`, since its documents are lenses files, which hold every template key.

Tests.
`TestConfigTemplate_DecodesAndIsSelfConsistent` extends its checks to every shipped layer of `lensSets()`, each in its own loop iteration:
the raw layer document's top-level keys are a subset of `lenses` and `fans`;
`CheckLoad` passes over a temp anchor whose `lenses.yaml` is `lens_set: <name>\n`;
in the `Load` of that anchor, each lens the layer defines resolves to the layer's prose, every other lens and every fan the layer lacks resolves to the base template's value, and no lens text carries `ignore `;
each layer lens not in a `review-*` or `fix-*` fan of the base template ends with the emphasis line, the exemption computed from the base fans only, so a lens a layer defines but the base lacks always ends with it, even when the layer adds it to a `review-*` fan;
each fan the layer defines starts with the base fan of the same name, exactly and in base order, and every later entry is a name the base fan lacks;
and, for `go`, each of the five lenses in the `Load` under `lens_set: go` equals the base template's, pinning the verbatim move at this card's gate.
The test's `//testtiming:keep` reason gains the per-layer load and superset checks.
`TestModules_EveryTemplateKeyClassified`'s `wantLocalDenied` for `lenses` becomes `{"fans", "lens_set", "lenses"}`.

Docs: `internal/lenses/doc.go` gains a `# Language layers` section: the layer files under `sets/`, the per-name replace-or-add rule, the base-only selector and its refusals, the superset rule for a layer fan, and that lyx picks no layer itself;
its `# Guard keys` section names `lens_set`, and its `# Reconcile` and `# Load` paragraphs say a file is judged against, and a missing name filled from, the effective template: the base with the picked layer laid over it.
`docs/overview.md`'s `internal/lenses/` tree line adds the language layer a hub picks.

**ImpactSummary:** `Config` and the template gain one key every consumer decodes through `lenses.Load`; `configsync`'s `lensesFixture` round-trips the new field as `lens_set: ""`, equal to the template, so the rows whose fixture is a lenses file keep their results, and the move rows' `burler.yaml` fixtures come from `burlerFixture`, which holds only `lenses` and `fans`, so the rows asserting `board:burler` absent keep theirs, and the one registry assertion naming lenses' denied keys is updated here; the build fingerprint now covers the layer files, so the build stamp changes once for every hub at landing.
