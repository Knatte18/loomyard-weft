```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Per change, the set found:**

1. **New `draftLayer` value from `ComputeLayers`.**
   - Callers: `ListTasksBrief` at `internal/boardengine/store.go:658` and `RenderOrder` at `store.go:669`, plus `renderReadme`. All pass the layer string through without matching on it, so none breaks. Covered.
   - `internal/boardcli/cli_test.go:261-267` only checks that the `layer` key exists. It is not affected.
   - No test outside `boardengine` pairs a draft status with a `Ready` expectation. Covered.
   - The `lym board list` layer field is documented at `doc.go:178`. Covered.
2. **`RenderOrder` bucket order.** `doneLayer` moved from 28 to 29 and nothing else reads the map. Covered by `layer_test.go` (the `d1` row).
3. **`layerSection` and the Layer A meaning line.**
   - The new strings are asserted in the goldens in `render_test.go`, and `doc.go:178` matches.
   - Searches of `internal`, `cmd`, `plugins`, `contracts` and `docs` (the `Waits only on` / `Independent` search covered the whole tree outside `_lym`/`.lym`) found no other copy of the old string. Covered.
4. **`standingCell` in `render.go`.**
   - A draft member falls through to `layerSection(member.Layer).name`, which returns `Draft`. This matches the acceptance criterion and the `T5` row in the golden. Covered.
   - Its doc comment, `render.go:442`, says "Done once finished, the run's place while a run holds it, Ready, Note, or the name of its dependency layer". It does not name Draft or Independent. This is not a defect, because "layer" covers them.
5. **Stale descriptions of the Tasks split.** One finding, below.

**Findings**

- **LOW, consistency**: two `render.go` comments still list the Tasks subsections without Draft.
  - `internal/boardengine/render.go:4` says "Tasks split into Running, Ready, dependency layers and Independent".
  - `internal/boardengine/render.go:238` says "Tasks split into Running, Ready, dependency layers and Independent".
  - Both now disagree with `doc.go:172`, `layer.go:3` and `render.go:317`, which name Draft. The diff edited the comment at `render.go:317` in the same file but missed these two.
  - Fix: append ", Independent and Draft" to both lines.

No other member of the set is left wrong. The diff stays out of `docs/research` and the link allowlist (the diff stat lists six `boardengine` files). The diff changes no stencil.

**Tool calls (4):**
1. Grepped the tree for the old meaning strings and "Independent", because my context lacked other copies of them.
2. Grepped for `ComputeLayers`/`RenderOrder`/`.Layer` consumers across the Go files, because my context held only the `boardengine` ones.
3. Grepped `store.go` and for draft-and-layer pairings in the tests, to find remaining consumers.
4. Read `store.go:650-680` and the `boardcli` layer assertions, to confirm that no consumer matches on a layer value.
