```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checking that every claim the fix added or rewrote holds against the code at HEAD.

no findings

I checked these claims:
- **Engine-name constants:** the engines' own name constants are unexported. Holds: `engineName` is `"claude"` in `internal/shuttleengine/claudeengine/claudeengine.go:28` and `"codex"` in `internal/shuttleengine/codexengine/codexengine.go:24`.
- **Model fields:** `ModelChoice` carries `Engine`, `Model`, `Effort` and `Version`, and a version pin or a Claude effort makes Codex's `Prepare` refuse the spec. Holds: `codexengine.validateChoice` refuses a non-empty `Version` and passes `Effort` to `validateEffort` against `codex.yaml`.
- **Allowlist precedent:** the `internal/runfsm.Order` entry is a "later card removes this entry" precedent. Holds at `cmd/lym/testonlycode_linux_test.go:59`.
- **Method entry key:** `internal/burlerengine.Engine.SeatTables` is a valid key form. Holds: `functionName` builds `Type.Method` with the pointer dropped.
- **`TestMain`:** it mirrors `internal/seatengine/testmain_test.go` by calling `gitkit.HermeticGitEnv()` and then `os.Exit(tmuxkit.Main(m))`. Holds.
- **`seatSpec`:** it copies `seat.ModelChoice` whole, so clearing `Model`, `Effort` and `Version` in `Seat.Specs` is what keeps them out of the spec. Holds at `internal/seatengine/prompt.go:229`.

Tool calls:
- One grep for the engine-name constants, the `ModelChoice` fields and `validateChoice`, because my context had only the reviewer's description of them.
- One grep of `testonlycode_linux_test.go` for the method-key form; it found no example.
- One `sed` read of `functionName` in that file, to confirm the `Type.Method` key form, which my context lacked.
