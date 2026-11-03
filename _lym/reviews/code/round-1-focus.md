---
round: 1
focus:
  - "The decision record's Goal names Sketch decisions D4, D5, D6, D7, D8 and D13 and the parent decision on the Weft checkout as a Codex writable root; its other sections are empty. Read the Sketch with `lym board get P2 --record` and settle whether `internal/shuttleengine/codexengine` implements each named decision, and whether any file in the diff implements a decision outside that set."
  - "Settle how the Weft checkout is passed to Codex as a writable root in `command.go` and `config.go`, and whether anything still routes the assistant report path through a writable root, which the parent decision replaced."
  - "`internal/fslink` (`fslink.go`, `fslink_linux.go`, `fslink_windows.go`) and `pattern/PATTERN-fs-links.md`: settle whether the new link behavior is licensed by a named decision, matches on both platforms, and has test coverage for what it adds."
  - "`internal/modelspec` (`load.go`, `registry.go`, `modelspec.go`, `template.yaml`) and `contracts/specs/llm-model-spec.md`: settle whether the loader and registry changes keep existing model-spec behavior and whether the spec text agrees with the code."
  - "`home.go`, `rollout.go`, `events.go` and `scrub.go`: settle failure paths of home creation, rollout discovery and event parsing (missing files, partial lines, cleanup on error), and whether secrets are scrubbed as the Sketch's constraints require."
  - "`internal/shuttleengine/engine.go`, `run.go` and `internal/shuttleengines/shuttleengines.go`: settle whether the seam changes are minimal, keep existing engines' behavior, and are covered by `startup_test.go` and the seam enforcement test."
  - "Read `git log` over the range for `Pattern-Departure:` trailers and settle whether each names an entry that forbids the way the Sketch asks for; check that `PATTERN.md` and `pattern/` changes land any declared entries."
---

Focus follows the diff's largest new surface, the `codexengine` package, then the shared packages it touches.
