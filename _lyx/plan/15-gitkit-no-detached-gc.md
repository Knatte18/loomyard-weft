# Card 15 — gitkit-no-detached-gc

**Edit:**
- `internal/gitkit#HermeticGitEnv`
- `internal/gitkit/gitkit_test.go`

**Prosa:**
- `internal/gitkit/doc.go`

**Intent:** D9: fixture git never gcs in the background.
The neutral config's `[gc]` section gains `autoDetach = false` beside `auto = 0`, so any gc that still runs is foreground and waited; the template repositories keep their local keys.
`TestHermeticGitEnv_QuietAndPinned` gains a `git config gc.autoDetach` read asserting `false`.
The hermetic variables reach every `lyx` child a test runs through `lyxbin.Run`, which inherits the process environment, and reed's clean environment already passes them through; the doc's hermetic paragraph says so.

**ImpactSummary:** One config line in the neutral file every git-spawning test package installs; no caller changes.
