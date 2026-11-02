1. The question the next judge must answer.

Why does the Darn-Write seat fill of stencil `darn-template-write` refuse with "unfilled top-level marker(s): subagent_model" although `DarnTable` sets that value, and what change makes the seat fill succeed?

2. The facts.

- Envelope `_lym/shed/landing-fixer-base-skills/drive-reports/drive-envelope-20261102-193422-b444.json`: kind `producer`, step `0bcd608ba4eaef96`, `transient` empty, `history_length` 4, `first_producer` empty.
- Trace `.lym/shed/landing-fixer-base-skills/steps/0bcd608ba4eaef96.trace.log` line 19: "shed: step refused" with the same error text.
- Stderr log `.lym/shed/landing-fixer-base-skills/steps/0bcd608ba4eaef96.stderr.log` is empty.
- Error path: `shedadapters: Darn-Write (seats): seat run: seatengine: seat "": fill stencil "darn-template-write": stencil: unfilled top-level marker(s): subagent_model`.
- `internal/darnengine/spec.go:77` requires `SubagentModel` non-empty; `spec.go:116` puts `"subagent_model": in.SubagentModel` into the table `Values`.
- `internal/loomcli/wiring.go:916` passes `loomCfg.SubagentModel` into `wireDarn`; `internal/loomcli/darnwiring.go:250` forwards it to `DarnInputs`.
- `internal/loomengine/config.go:1126` refuses an empty `subagent_model` at load, so a run past load carries a non-empty value.
- The marker is written in `contracts/stencils/darn/darn-template-write.md:63` as `{{.subagent_model}}`, and the comment at line 12 calls it required.
- Commit `2d319fe5` added `subagent_model` (config, stencil, darnengine) in one change.
- `.lym/config/loom.local.yaml` holds only `code_hard_max_bounces` and `code_max_bounces`.
- The hub file `_bolt/_lym/config/loom.yaml` holds only `discussion_reviewer_fan`; `subagent_model` comes from the template default in `internal/loomengine/template.yaml:6`.
- `lym config reconcile` (dry run) reports no added or removed keys for module `loom`, so the config layer is not the missing piece.
- `lym config loom --print` returns `{}`, which I did not explain.

3. What I ruled out, and how.

- Empty config value: `DarnTable` and `LoadConfig` both refuse an empty value, and the trace shows no refusal from either.
- Stale or missing config key: the dry-run reconcile reports nothing to add for `loom`.
- Transient or interrupted failure: the envelope has `transient` empty and `interrupt_policy` reinvoke with no re-step recorded, and the error is deterministic at the fill.
- Stderr cause: the stderr log is empty.
- Not checked: the seat fill code that reads the stencil's top-level markers and matches them to `Values`; I did not open it, so the mismatch is not located.

4. Hypothesis (unverified).

The stencil fill's top-level marker check looks up `subagent_model` in a table the seat fill does not receive, or uses a key form that does not match `Values`, so a plain re-launch fails the same way. A stencil, seat-engine, or stencil-check defect after commit `2d319fe5` is the likely cause; a fix needs code read and judged, so the next judge should read `internal/seatengine` fill code and the stencil marker check, and confirm with a test that renders `darn-template-write` with `Values` from `DarnTable`.
