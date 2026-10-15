---
verdict: APPROVED
---

## Scope against the board entry

The decision record covers every scope point of `engine-config-seams` and nothing another entry owns:

- `claude.yaml` owned and loaded by `claudeengine`, layered and registered in `configreg`, holding the binary path, the prompt-cache TTL and its per-role map, and the Agent and AskUserQuestion denies: D1, D2.
- `deny_python` stays neutral in `shuttle.yaml`, and the old keys move out of `shuttle.yaml`: D3, D4.
- One neutral shell-guard table (pattern, steer, notice, switch) in a leaf package, rendered by `claudeengine` as PreToolUse hooks, read by the no-denied-recovery test instead of its hand list: D5, D6, D7, D10.
- Engines declare the environment keys to scrub, replacing `CleanClaudeEnv`: D8.
- An opaque provider tag set by shuttle, a `SessionNamer` per provider, no session-name repair for a tag with no namer: D9.
  The empty tag mapped to the `claude` namer covers strands recorded before the tag and does not contradict the entry, since `claude` has a namer.

The Out list matches the entry and its design points: Codex rendering and `codex.yaml` stay with `codex-engine`, timing-key overrides wait, the `lyx` hook verb is not built, usage reading stays with `engine-usage-readers` (which depends on this task), role engines stay with `engine-model-choice`, and `spawn-audit-log` lands after this task as the entry says.

No parent commits are listed.

## Overlaps

- `creel-core` (row 12, own row 3): both change shuttle runner and reed engine construction, `creelcli`'s sites among them, and `creel-core`'s record names this run's provider tag.
  Both records agree that whichever lands second adapts its construction sites to D2 and D8; `creel-core`'s default-engine send to a strand without a record and this run's empty-tag namer mapping govern different behaviour.
  Landing order: `creel-core` is further along and lands first; this run takes its changes at Merge-In.
- `engine-model-choice` (row 3, own row 3): both change `internal/shuttleengine`, that task `Spec` and `OpenSpec` through an embedded `ModelChoice`, this one `Config`'s Claude fields and `shuttleengines.New`'s inputs.
  The decisions do not touch the same behaviour or the same PATTERN entry (`PATTERN-model-choice-mapping` against `PATTERN-shell-guard-table` and the provider-seam background); its record already names this run as a parallel `shuttleengine` change whose second lander adapts.
  Landing order: a tie on row 3.
- `lighter-test-rules` (row 7): no overlap; its Out list keeps assertions on Go-built text such as a deny notice, which D6 keeps byte-identical.
- Shared bookkeeping: `cmd/lyx/testdata/test-budget.yaml` (new `shellguard` row), `docs/overview.md` and new PATTERN index lines, named here only.
