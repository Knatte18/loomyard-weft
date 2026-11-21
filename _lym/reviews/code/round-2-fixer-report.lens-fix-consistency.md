```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F15–F18 change set against its siblings (docs, stencils, stand-ins) for stale cross-references.

**Findings**

1. **LOW** — `internal/darnengine/doc.go:90-91`, package doc. The paragraph still says "The caller also tells DarnTable the measuring run's gate call". Since F15, only a gate-fixed writer needs the call. `DarnTable` now refuses an empty `MeasureCall` only when `GateFixed` is set (`spec.go`). The sibling `MeasureCall` field comment and the `DarnTable` doc were updated to say so, but this package doc was not. A reader would think every caller must supply the call. Suggested edit: "With a cold fixer the caller also tells DarnTable …".

2. **LOW** — `internal/darnengine/doc.go:93`, "DarnTable refuses an empty required field by name". The required list is now conditional: `MeasureCall` joins it only for a gate-fixed writer. This is the same stale-doc twin as finding 1, one line below it.

**Checked, consistent**
- `contracts/stencils/darn/darn-fragment-make-change-gated.md:7`: the header names the gated fragment only, so it still holds.
- `internal/shuttleparity/seats.go:212`: the stand-in passes a `MeasureCall` while `GateFixed` is unset, so `DarnTable` ignores the value and the stand-in is harmless.
- `MeasureCall`'s comment in `tiers.go` and the `MeasureCall` paragraph in `gateslot/doc.go` agree on the covered-tier rule.
- `wiring_test.go`'s "no gate.yaml" row was updated to match the ungated build.

Further tool calls (3 of 6):
1. Grepped `internal/darnengine/doc.go`, `internal/loomcli`, `docs`, `contracts/stencils/darn`, `internal/shuttleparity` and `internal/gatecli/doc.go` for measuring-call mentions: my context did not hold the twins of the changed text.
2. Read `darn-fragment-make-change-gated.md:1-14` and `internal/shuttleparity/seats.go:205-216`: to check the stencil header and the stand-in against the new conditional requirement.
3. Read `internal/darnengine/doc.go:82-96`: to see the package doc paragraph that grep matched.
