# Card 10 — shed-driver-pattern

**Create:**
- `internal/sheddriver/seam_enforcement_test.go`
- `pattern/PATTERN-shed-driver.md`

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-told-geometry.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Pin `internal/sheddriver` as the one Go driver of shed runs (D9) and as a told-geometry engine (D1), now that its code has landed.

`seam_enforcement_test.go` holds `TestToldGeometryInvariant_AllowlistOnly`, modelled on `internal/battenshed/seam_enforcement_test.go`.
It calls `scankit.AssertImportAllowlistNoStale` over the package's production imports and separately reports, by name, any direct `internal/lyxcwd`, `internal/loomengine`, `internal/loomshed` or `internal/*cli` import, the four exclusions D1 states.
The allowlist is the set the package uses after cards 2 to 8, each with a one-line reason where it is not obvious:
- `buildvcs`, `configengine`, `editdirective`, `lock`, `logger`, `modelspec`, `output`, `parentdirective`, `pauserecord`, `segmentcolor`, `shedengine`, `shedverbs`, `shuttleengine`, `stencil`, `stencilstore`;
- `shedrun`, for the driver-kind vocabulary and its validator only, since every `shedrun` path constructor takes a `*lyxcwd.Location` the package can never hold;
- `gopkg.in/yaml.v3` for the verdict and config decoding.
The implementer runs the test and makes the list exactly the imports present, so the no-stale check passes; a needed import outside the four exclusions is added with its reason, never a denied one.

`PATTERN.md` gains, under "Shed and loom", after `PATTERN-shed-verb-set`:

```markdown
- `PATTERN-shed-driver` — Driving a shed run in Go, deciding its stops or spawning a tackler: only `internal/sheddriver`, told its paths and seams. (test) — [background](pattern/PATTERN-shed-driver.md)
```

The overview sits within a few bytes of its headroom line (`HeadroomBytes` in `internal/pattern/check.go`), so the card gate's `lyx loom lint-pattern` fails on any net growth past it.
The card shortens three entries whose detail their background files already state, so the overview ends at or below the line:
- `PATTERN-verified-tree` drops ", never on a dirty tree, skipping only on a record of HEAD's tree";
- `PATTERN-agent-name` drops "; the spawner owns the role prefix, config a later seat's suffix";
- `PATTERN-test-speed` drops "of four, `llm` files alone reach an LLM, inject clocks" down to "lowest fitting tier, injected clocks, `t.Parallel` by default".
Each kept clause stays true against its background; when `lyx loom lint-pattern` still reports bytes to cut, the implementer shortens another entry the same way, never the new one.

`pattern/PATTERN-shed-driver.md` states:
- the rule and its enforcement: the seam test pins the imports, while "no `<module>cli` or producer runs a step-until-stop loop of its own" is review discipline, as `PATTERN-shed-verb-set`'s first clause is;
- the seams a recipe fills, their unwired behaviour and D2's bound;
- that batten's InnerRun wait, which polls a child run it does not step, is outside it.

`pattern/PATTERN-told-geometry.md` adds `sheddriver` to its bound-packages list.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/sheddriver` row rises from 19 to 20.

**ImpactSummary:** one PATTERN entry added and three shortened, one background added and one extended, and a new seam test; no production code changes.
