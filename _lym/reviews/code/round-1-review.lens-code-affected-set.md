```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each output or behaviour the diff changes, I looked for everything that reads, asserts or describes it, and checked whether the diff left any of it wrong.

**Affected set per change**

- **Chair-timeout Stuck reason text** (`internal/shedadapters/multillm.go:210`, now "timed out and was stopped"):
  - Readers and assertions: only the three timeout rows of `TestMultiLLMProducer_MapsTheLeadResult` (`internal/shedadapters/multillm_test.go:360,367,376`). The diff updates all three.
  - No other Go file, spec, stencil, skill or doc matches "session timed out" or the old way-forward text. No parser of the reason exists anywhere in the repository.
  - Covered.
- **`timeoutWayForward` constant** (`internal/shedadapters/multillm.go:227`):
  - The only reference is the `mapOutcome` call site above. `internal/shedrecipe/entries_command_test.go:62` declares a local constant with the same name, but it is unrelated test-only text.
  - Covered.
- **Comment on `timeoutWayForward`** (`internal/shedadapters/multillm.go:226`, "The seat engine stops a timed-out chair before the step ends"):
  - The behaviour it describes is in `internal/seatengine/engine.go:475-477` (`join` stops a chair with `OutcomeTimeout` when the step was not ended) and in `join`'s doc at `engine.go:430`.
  - A failed stop returns an error, so this reason is never reached. `Call` goes through `errorExit` (`multillm.go:141-143`).
  - Covered.
- **"A resume starts the row over in a fresh session"** (the new way forward plus `internal/shedadapters/doc.go:34-36`):
  - The resume path is `Call` → `Probe`. A stopped chair is not live, so `Probe` stops live advisors and returns an empty `LiveTable` (`internal/seatengine/resume.go:33-66`), and `Call` then takes the fresh `Run` path (`multillm.go:122-144`).
  - The crash-resume path cannot resume the stopped session. Stop leaves an already-terminal timeout record untouched (`internal/shuttleengine/stop.go:46-60`), and `resumableAfterInterruption` returns false for a timeout outcome (`internal/shuttleengine/crashresume.go:28-41`).
  - Covered.
- **Package doc** (`internal/shedadapters/doc.go`, the MultiLLMProducer bullet):
  - The bullet's closing line, "Every such Stuck reason ends with the way forward of running `lym loom resume`", stays true: the new text still names `lym loom resume`.
  - `internal/seatengine/doc.go`'s stop rule (lines 52-55) does not mention the timed-out chair stop. This is not stale, because the diff changes no seatengine behaviour.
  - Covered.
- **Repository rules:**
  - `PATTERN-refusal-way-forward`: the reason still ends in a `way forward:` clause, the text names no denied step, and the tests still reach and assert the message.
  - `PATTERN-code-conventions`: the wording stays one named constant.
  - Comments are on their own lines and self-contained.
  - Covered.
- **Declared PATTERN changes:** the directive declares none, so there is nothing to check.
- **Commit messages and departures:** the range is one commit, fd7f07747. It has no `Pattern-Departure:` or `Loosened-Check:` trailer. The test changes swap expected strings for new ones and loosen nothing.

**Findings:** none.

Tool calls:
1. Repo-wide grep of Go, md, tmpl and yaml files for the timeout wording, plus a read of `seatengine/doc.go`'s stop lines: the manifest's grep covered only some wording variants and did not include seatengine's package doc.
2. Grep of the `_bolt` specs, stencils, `docs` and `PATTERN.md` for the old wording, and a read of the `PATTERN-refusal-way-forward` background file: the manifest named the pattern but had not read it.
3. Grep of `contracts/specs`, `contracts/stencils` and `plugins/ly/skills` for re-attach or timeout wording: the refusal pattern names these as the scanned surface, and the manifest had not searched them.
