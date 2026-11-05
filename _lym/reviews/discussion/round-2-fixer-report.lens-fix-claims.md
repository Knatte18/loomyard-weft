```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked every code-facing sentence the fix added or rewrote against HEAD.

Claims that hold:
- In each `rollout-evidence.txt` total that carries `total_tokens` (context-parity, Codex CLI 0.164.0 per `report.md:4`), `total_tokens` equals input plus output.
- Codex implements no `TranscriptReader`.
- `RecordReaders` hands out `Transcripts` and `Pricer`.
- `Bolt.Commit` stages every change.
- `boardCriticalSection` spawns `spawnSync`.
- `Claims` refuses paths that contain `/`.
- `create.go` writes the beam flag together with the owner and the colour.
- "Unreleased" maps to status `draft`.

Findings:

1. MEDIUM — D10, "replays each store file by its design-file rule: a file origin does not hold lands".
   The design-file rule itself refuses a nested path. `applyDesignFile` refuses any `file.Path != filepath.Base(file.Path)` as "not a plain file name" (`internal/boardengine/replay.go:572-574`). Widening `Claims` alone still leaves every `raddle/<slug>/…` file refused. D10 and the new Constraints Fact name only `Claims` as the thing that refuses a `/` today.

2. LOW — the new Constraints Fact on ended runs and D9 Bound, "spares only a run that entered Finalize".
   `finalizeEnteredRead` also spares a run when the prime's status is `done` or its rows name Land or worktree teardown (`internal/fabricengine/replay.go:275`). So "only" is too strong. It doesn't change D9's step 7 reasoning, since raddle reaches none of those before step 7.

3. LOW — Constraints, the Codex fact the fix now sits beside ("neither carries a cache-write count").
   The evidence file the new fact cites shows a rollout total carrying `"cache_write_input_tokens":0` (`docs/research/codex-astra-context-parity-bench-2026-10-27/data/rollout-evidence.txt:2-5`). That contradicts the adjacent fact and D12's "the rollout carries no cache-write count".

Tool calls:
- Read `internal/shuttleengines/shuttleengines.go:210-244`: the context lacked how `RecordReaders` fills `Transcripts` and `Pricer`.
- Grep in boardengine and codexengine, plus `board.go:74-90`: the context lacked the release (draft) representation, `boardCriticalSection`'s lock, and whether Codex has a `TranscriptReader`.
