```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set — what the diff's changes force outside the lines it touches.** The set was covered for every change below except the first finding (F1).

**Per change: set found, covered or not**

1. **`Table.Validate` and `Table.CheckMarkers` gain an engine parameter.**
   - Callers found: `seatengine/engine.go:230`, `resume.go:30,94`, `prompt.go:97,129`, `shedrecipe/entries_multillm.go:98` and `loomcli/darnwiring.go:272`. All are updated.
   - `readStencilSet` and `validateStencil` are internal to `seatengine` and updated. The `seatengine` test callers are in the diff.
   - Covered.

2. **`DefaultEngine` is set at every place that builds a Geometry or Env.**
   - Set at `burlercli/wiring.go:162,234`, `loomcli/wiring.go:565,741,910`, and `burlerengine/engine.go:56`, which passes it on to `seatengine`.
   - `hubgeom.BurlerGeometry` and `standalonegeom.BurlerGeometry` return it empty, and their only callers set it.
   - The other `shedrecipe.Env` literals, `battencli/wire.go:919`, `loomcli/landingdeps.go:437` and `testkit/envkit`, leave `DefaultEngine` empty. None of them builds a `seatengine`, and no engine block ships yet. Not a defect now; see F1.

3. **`RelPath` returns `name.md` for a name holding "/".**
   - Callers: `stencilstore.Path`, `reconcile.go:122,159,444`, `fabricengine/stencilhistory.go:32`, `stencilcli/promote.go:67` and `stencilcli/diff.go:160`. All take the new shape and none depends on the old family prefix.
   - Names are registry-driven, and `contracts/stencils/engine/` does not exist yet. `registry_test.go` was extended to walk it, so the new shape is covered.

4. **Engine-block resolver (`ReadSet`) is the only reader of includes.**
   - Only `seatengine` and `burlerengine` call `stencil.FillWith` with blocks. Every other producer uses `Read` plus `Fill` on stencils with no engine include.
   - Callers of `burlerengine/prompt.go` stencils elsewhere: none. The `burler-step-*` and `burler-cluster-*` names appear outside `prompt.go` only in `burlerengine/doc.go` and a Warn string at `fix.go:141`.
   - No second resolver is left. Covered.

5. **`burlerengine` passes the half's engine.**
   - `Review` and `Fix` pass `seatEngine(opts.Review)` and `seatEngine(opts.Fix)`.
   - `doc.go` is updated, and the `prompt.go` header and per-function comments match. Covered.

6. **`codex-file` fallback and carry-over (`render.go`, `target.go`).**
   - `seam.go:41` consumes `ReRender`, which now holds `CLAUDE.md` for `codex-file`. Covered.
   - `shipped.json` has entries on `claude-file` only (lines 8 and 16), so no shipped entry reaches `codex-file`. This matches D12.
   - The `heddlecli/verbs.go` help text names only `claude-file`. No change is forced.
   - The `heddleengine/doc.go` update is present. Only `pattern/PATTERN-durable-vs-ephemeral-state.md` mentions `codex-file`/`AGENTS.override`, and that line is unaffected.

7. **`PATTERN.md` gains `PATTERN-engine-blocks`.**
   - The text matches the record's declared text (D11, including "adds:" being dropped). `PATTERN-fs-links` is untouched, as D13 belongs to another part.
   - Covered.

8. **Commit messages.** The three messages match the diff. There is no `Pattern-Departure` trailer, and none is needed.

**Findings**

- **F1 — LOW, consistency.** `shedrecipe.Env` literals in `internal/battencli/wire.go:919` and `internal/loomcli/landingdeps.go:437` leave `DefaultEngine` empty. A MultiLLM row built from them would hit the empty-engine refusal in `stencilstore/engineblock.go:885` once any engine block is included. Nothing consumes it today, and the loom wiring (`loomcli/wiring.go:910`) is the one that builds seats. This is a latent gap, not a present defect.
  - **Fix:** none needed now; set it when a battencli row first builds seats.
- **F2 — NIT, consistency.** `docs/shared-libs/stencil.md` describes the include and block rules and is not updated for engine blocks (the `engine-` prefix and per-engine resolution). I did not open it, so I cannot say its text is now wrong.
  - **Fix:** check whether it states the include-resolution rule, and add a sentence pointing to `stencilstore.ReadSet` if it does.

Tool calls (all Bash, 5 of 6):
1. Grep for Geometry and Env literal constructions, to check `DefaultEngine` is set everywhere. My context lacked the full caller list.
2. Grep for `RelPath` and `ReRender` consumers, and for `codex-file` in docs. My context lacked these.
3. Grep for other users of the burler instruction stencil names. My context lacked them.
4. Grep for `claude-file` and `codex-file` in `shipped.json` and in non-heddle Go code, to check the shipped-entry rule and other consumers.
5. Sed over the `loomcli` and `battencli` Env literals, to see which struct line 910 belongs to and whether the other literals set `DefaultEngine`.
6. Grep for `burlerengine.New` and `seatengine.New` callers and for docs on includes. My context lacked these.
