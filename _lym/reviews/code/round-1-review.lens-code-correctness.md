```yaml
lens: code-correctness
outcome: reported
```

Embedded `ModelChoice`, so `s.Engine` is the right field.

Lens: code-correctness. The diff holds up on every case I ran. I found no defects.

**Per function or branch (case run, result)**

- `stencilstore.ReadSet`, stencil with no include: returns the body and an empty map, same as before. Correct.
- `ReadSet`, stencil name missing: returns the `Read` error. The seat wrappers still prefix it with the seat label. Correct.
- `ReadSet`, stencil with `engine-foo`, engine `codex`, no file: returns `*MissingEngineBlockError` naming block, engine and absolute path. Correct.
- `ReadSet`, include with the `engine-` prefix and an empty engine: returns a way-forward error naming the stencil and block. Correct. A seat whose model choice names no engine, with `DefaultEngine` unset, is refused instead of reading a wrong block. That is intended.
- `ReadSet`, ordinary include: read through `Read`, so the old behaviour is kept.
- `ReadSet`, include name `engine-` followed by nothing: block is `""`, the path is `engine/<e>/.md`, and it reports as missing. Harmless.
- `readInclude`, a non-ENOENT read error: wrapped with `%w`. Correct.
- `RelPath`, name holding `/`: returns `name.md` and skips the family split. Existing names have no `/`, so they are unchanged.
- `seatengine.readStencilSet`, `validateStencil`, `CheckMarkers` and `composeSeatPrompt`: all pass `Seat.stencilEngine(default)`. `Seat` embeds `ModelChoice`, so `s.Engine` is the model choice's engine. Validate, the marker check and composition read the same set, and I found no site that still reads blocks engine-blind. Correct.
- `Prompts`, `Run`, `Probe` and `Resume` pass `geom.DefaultEngine`. Correct.
- `burlerengine.seatEngine(choice)`: the choice's engine, else `geom.DefaultEngine`. `Review` and `Fix` pass `opts.Review` and `opts.Fix`, the same choices set as the seat's `ModelChoice` (`review.go:63`, `fix.go:83`), so the seat and its instruction files agree on the engine. Correct.
- `fillInstruction`: `ReadSet` then `FillWith`, with the error wrapping kept. The prior behaviour is the same for non-engine stencils, because `FillWith` with nil includes equals the old `Fill`/`FillOptional`. Correct.
- `composeReviewPrompt`: the cluster-rules call moved ahead of the review template read. Only the order of errors changes, not the result. Correct.
- `focusDirectiveBlock`, empty path: returns `""` without reading. Correct.
- `forkReadingBlock` and `selfCheckBlock`, empty fan: return `""` without reading. Correct.
- Wiring (`burlercli` hub and standalone, `loomcli` `burlerGeom`, seats geometry, `shedrecipe.Env`, `darnwiring` `CheckMarkers`): all set `DefaultEngine` from `shuttleCfg.Engine`. Correct. `Env.DefaultEngine` reaches `multiLLMEntry` through the `loomcli` env I saw at `wiring.go:910`. I did not trace whether other `Env` constructors, such as the standalone path, set it. If one leaves it empty, an engine-block include there would be refused.
- `withCodexFile(only)`, `only` nil or empty: returned as is, so every file target renders. Correct.
- `withCodexFile(["claude-file"])`: returns a clone with `codex-file` added. The caller's slice is not mutated. Correct.
- `withCodexFile(["claude-file","codex-file"])`: unchanged, no duplicate. Correct.
- `fileText` entry filter, entry targeting both `claude-file` and `codex-file`: the single `||`-style condition admits it once, so no duplicate text. Correct.
- `fileText` entry filter, shipped entry aimed at `claude-file`: skipped for `codex-file`. Correct.
- `fileText` entry filter, repository entry aimed at `claude-file`: carried into `codex-file`. Correct.
- `fileText` `codex-file` opening, `AGENTS.md` and `CLAUDE.md` both present: `AGENTS.md` is taken, then `break`. Correct.
- `fileText` `codex-file` opening, only `CLAUDE.md`: `CLAUDE.md` is taken. Correct.
- `fileText` `codex-file` opening, neither file present: both reads give ENOENT and the loop continues. With no entry texts there is no file, as before. Correct.
- `fileText` `codex-file` opening, read error other than ENOENT: returned, naming the file. Correct.
- `targetTable` `codex-file` `ReRender`: gains `CLAUDE.md`, so a `CLAUDE.md` change re-renders it. Correct. `.claude/CLAUDE.md` is not listed, which matches D12 naming the anchor's `CLAUDE.md` only.

**Calls made**

1. `grep` for the heddle layer constants: to confirm a lym-override entry keeps `LayerShipped`. Shipped-layer entries are filtered by `entry.Layer`, and `OverrideLayer` is a separate constant from the layer values.
2. `grep -n "^func Read..."` in `stencilstore.go`: returned nothing, so I did not check whether `Read` adds checks that the direct `os.ReadFile` in `readInclude` skips.
3. `grep` of `opts.Review`, `opts.Fix` and `RoundModels` in `burlerengine`: to confirm which choice feeds the engine. Both halves pass their own choice.
4. `sed` of `internal/seatengine/table.go` plus a `grep` for `ModelChoice`: to confirm `Seat.Engine` is the embedded choice's field.
