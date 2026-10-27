```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD (fix-claims lens).

**Claims checked:**
- **Holds:** "every `internal/state` read and write calls `os.MkdirAll` on the record's directory". `WriteJSON`, `WriteFile`, `ReadJSON` and `UpdateJSON` (`internal/state/state.go:147-151`) all do. `UpdateState` goes through `UpdateJSON` (`internal/dobbyengine/state.go:96-97`).
- **Holds:** "a tick after a removal recreates `.lym/beam`, and the stub holds `state.json` and a lock file". `tick` opens with `UpdateState`, and the lock path is `path + ".lock"` in the same directory (`internal/dobbyengine/state.go:114`).
- **Holds:** "a tidy it settled is stopped again". `settleTidy` stops the strand before it clears `State.Tidy` (`internal/dobbyengine/tidy.go:201-205`).
- **Holds:** the Decision 2 test and its uses, and the Decision 5 Windows exception, agree with the rest of the record and make no claim about the code that fails.

**Finding:**
- **LOW.** Changed text, Decision 1: "and one it settled is stopped again, which removes nothing and returns nil once reed no longer tracks its strand."
  This holds only when reed's status probe answers.
  `removeStrandIfPresent` returns an error when `reed.Status()` fails (`internal/shuttleengine/stop.go`), so the second stop can log a failed-stop line.
  It can also remove a strand: `settleTidy`'s own stop may have failed, which `stopTidyStrand` only logs (`internal/dobbyengine/tidy.go:222-225`), leaving the strand tracked, and the exit's stop then removes it.
  So "removes nothing" is unconditional where the code makes it conditional.

Further tool calls:
1. `grep` over `internal/state/state.go`, `internal/dobbyengine/state.go` and `internal/dobbyengine/tidy.go`. My context lacked `UpdateJSON`'s mkdir, `UpdateState`'s path, and how the tidy settle stops a strand.
2. `sed` over `internal/dobbyengine/tidy.go:180-235` and `lockPath`. My context lacked the settle order and where the lock file lives.
