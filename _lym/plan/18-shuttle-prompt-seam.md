# Card 18 — shuttle-prompt-seam

**Create:**
- `plan:internal/shuttleengine#PassageReader` -> `type PassageReader func(boltDir, engine string) ([]string, error)`
- `plan:internal/shuttleengine#installedPassageReader` -> `var installedPassageReader atomic.Pointer[PassageReader]`
- `plan:internal/shuttleengine#InstallPassageReader` -> `func InstallPassageReader(r PassageReader) (restore func())`
- `plan:internal/shuttleengine#StandingPassages` -> `func StandingPassages(boltDir, engine string) []string`
- `plan:internal/shuttleengine/claudeengine#TestPrepare_AppendsStandingPassages` -> `func TestPrepare_AppendsStandingPassages(t *testing.T)`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#appendPromptText` -> `func appendPromptText(notice string, passages []string) string`

**Extends:**
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/heddleengine#Renderer.Passages`
- `internal/testkit/logcapture#Capture`

**Intent:** Deliver the `claude-prompt` entries into every Claude session shuttle launches in a hub (D4), through a process-wide seam so heddle knows no Claude flag and `claudeengine` parses no `heddle.json` (PATTERN-shuttle-provider-seam).

- `internal/shuttleengine` declares `PassageReader`, the standing passages for a hub's Bolt checkout and an engine name, and `InstallPassageReader`, an `atomic.Pointer` install returning its restore, as `fabricengine.InstallPushNotifier` does.
  `StandingPassages(boltDir, engine)` returns none for an empty `boltDir` (a session outside a hub) or with no reader installed; a reader error returns none and is logged at Warn naming the error and `lym heddle list` as the way to inspect the store, so a broken `heddle.json` never blocks a launch and lym's own sentences still reach it; for a file that does not decode, the error is card 14's refusal, which names restoring the committed file as the repair.
  Heddle's reader is `heddleengine.Renderer{}.Passages`, which reads `<engine>-prompt`; card 20 installs it, and `codex-engine` will read `codex` through the same seam when it is built.
- `Prepare` reads `shuttleengine.StandingPassages(c.dirs.Board, "claude")` (the told `Board` dir is the Bolt checkout) and passes them to `appendPromptText`, which keeps lym's notice and sentences on their one line as today and appends each passage after a blank line, in order.
  The one-line rule the package's `init` check applies stays on lym's own sentences; passages may span lines, since the file reaches Claude Code through `--append-system-prompt-file` and is echoed nowhere.
- A resume through the launch's own resume line reuses the file its launch wrote, so its passages stay; a resume that runs `Prepare` again, as an adopted session does, reads the store afresh.
- The two `doc.go` files describe the seam and the file's layout.

`TestPrepare_AppendsStandingPassages` (tier0, in `prepare_test.go`; not parallel, since it installs the process-wide reader and captures the logger) rows over an engine whose told dirs carry a temp `Board`:
with a reader returning two passages, the file holds lym's notice and sentences, a blank line, the first passage, a blank line, the second (that `Launch.ResumeCmd` names this same file is already pinned by `TestPrepare_AppendSystemPromptMatchesDenyNotice`, so the resume keeps the passages with no row of its own);
with no reader installed, the file holds lym's sentences only;
with a reader installed and an empty `Board` (outside a hub), the file holds lym's sentences only and the reader is never called;
with a reader returning an error, the launch succeeds with lym's sentences only and the captured log names `lym heddle list`.
`TestPrepare_AppendSystemPromptMatchesDenyNotice` runs with no reader installed and stays as it is.

**ImpactSummary:** One new seam read in `Claude.Prepare` and a re-signed private formatter with that one caller; with no reader installed the appended file is byte-identical to today's.
