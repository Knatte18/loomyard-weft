```yaml
lens: fix-claims
outcome: reported
```

Checked whether each claim the fix added or rewrote holds at HEAD.

Claims checked:
- `lym heddle list` shows every entry targeting `claude-file` or `orch-prompt`.
- The shipped entries' text names Read, Edit, Write and the gate.
- `parentdirective` renders its no-parent variant for an empty parent name.
- Codex's `FileAuthStorage::save` opens the file with truncate and write and renames nothing.
- Codex's logout removes the file through `storage.delete()`.
- `workspace-write` makes only the workspace, `/tmp` and `$TMPDIR` writable, and Codex keys a linked path by its resolved target.
- creel types posts into every strand, and notices and reminders into strands without creel hooks.
- The attempt token is `1`, then `1b`.
- D6's dropped-prompt ending.
- D1's bound when the lym process dies.

Findings:

1. MEDIUM. D6, "when Codex stays alive but its first turn never starts … no `SessionStart` arrives, the run ends at its timeout and the marker reads `timeout`."
   This is wrong.
   A launch that sets `RecordsSessionStarts` "starts only once the events file holds a session-start line … until both hold the run stays not started and the startup error names the missing line" (`internal/shuttleengine/doc.go:546-547`).
   `internal/shuttleengine/startup_test.go:190` asserts "a ready pane with no session-start line is not started".
   So the run fails at startup, not at its run timeout, and D1's marker reads `not started`, not `timeout`.
   The review's other point does hold: shuttle does not retry this case, since it relaunches only a start whose teardown ended died (`doc.go:533`).
   The sentence should say that the run fails to start, is not retried, and the marker reads `not started`.

2. MEDIUM. D1 bound, "a lym process that dies before writing the marker leaves the chair waiting until its own review timeout or until a resume's probe writes the marker." (The fixer took this claim from the review's fix.)
   The chair's review timeout is the run's deadline, and shuttle's Wait enforces it inside that same lym process (`internal/shuttleengine/doc.go:75`, `:86`).
   `lym shuttle await` has no deadline of its own, only a cap, after which the chair runs it again.
   So when the lym process dies, nothing enforces the chair's timeout: the chair waits until a resume's probe writes the marker, or until someone stops it.
   The bound should drop "until its own review timeout", or name what does end the wait.

3. LOW. D1, "the round's attempt token the review row passes (`1`, then `1b` on its retry)."
   The token is `strconv.Itoa(round)`, plus `b` on the second attempt (`internal/shedadapters/burlerreview.go:254-256`), so `1` and `1b` are round 1's values only.
   A reader could take them as fixed values; "`<round>`, then `<round>b`" would be exact.

Calls:
- grep `internal/shuttleengine` for `RecordsSessionStarts`: my context lacked shuttle's startup handling for D6's claim.
- grep `RecordsSessionStarts` in the shuttleengine sources and tests: to trace the startup criterion and its test rows.
- read `internal/shuttleengine/doc.go:540-556` and its retry lines: for the not-started and retry rules.
- grep timeout lines in `doc.go` and `await.go`: to find who enforces the chair's timeout for D1's bound.
- read Codex `manager.rs` at `rust-v0.164.0`: my context lacked the logout path behind D13's claim.
