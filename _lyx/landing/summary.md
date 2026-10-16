# lyx orch start --continue resumes the last orch session

After a reboot or a dead orch strand, `lyx orch start --continue` brings back the same orch conversation.
Before, that took `lyx orch start --adopt <id>` with the id dug out of `~/.claude/projects/`.

## What changes

- `orchengine.State` gains an optional `session_id`, the provider session id of the strand `State.Strand` names.
  `start` writes it at every fresh launch, plain or adopt.
  When `start` binds the state to a live strand other than the recorded one, it clears the id until the watcher reads that strand's id.
- The orch watcher reads the strand's session each tick and rewrites `session_id` when the id differs, so the id stays current across `/clear` cycles.
  It writes through the strand-checked save, so it never writes onto a strand a concurrent `start` just bound.
- `lyx orch start --continue` takes the existing adopt path with the recorded id.
  The envelope reports `prompt_source` `adopt` and a new `session_id` field.
- `--continue` refuses, before removing or launching anything, when `--adopt` or `--handoff` is also given, when the strand is live, or when no id is recorded.
  A shuttle resume-check refusal surfaces as the start error with the two ways forward (plain `lyx orch start`, or `--adopt <id>`), and a failed launch leaves the recorded id in place.
- Every fresh launch that saves an id other than the recorded one adds a hint naming `lyx orch stop` then `lyx orch start --adopt <previous id>`.
  A `--continue` launch, or an `--adopt` naming the recorded id, adds none.
  When the watcher-stopping hint also applies, the hint carries both, that one first.
- `lyx orch status` reports the recorded session id, empty when none.
- `internal/shuttleengine`'s session reading gains the session id its fold picked, so orch parses no transcript or provider path.
  The change is additive and read-only.
- `internal/orchcli/doc.go` and `internal/orchengine/doc.go` describe the flag, the record and the reboot recovery; `cmd/lyx/testdata/test-budget.yaml` is adjusted for the new tests.

## Design decisions

- `--continue` is an explicit flag, not the default for a dead strand: a dead strand also follows `lyx orch stop`, a crash or a wish for clean context, where resuming is the wrong default.
  The hint covers an operator who forgets the flag.
- The record is in the never-tracked orch state file, so it survives `lyx orch stop` and a reboot.
  A state file without the field reads fine; only `--continue` treats its absence as a refusal.
- In compact mode the id never changes, so the watcher writes nothing after the launch.

## Check by hand

- `--continue` resumes only an id orch recorded for its own strand, at launch or as the watcher read it.
  The watcher's half relies on Claude Code naming the parent session id in a subagent's or fork's hook lines (`internal/orchengine/watcher.go`, reading the newest signal of any kind).
  A version that names the subagent's own id there would make the watcher record it, and `--continue` would resume the subagent's session.
- Not run: a real reboot, or a real `/clear` cycle against Claude Code, to confirm the watcher records the post-clear id.
  If a version stops minting a new id on `/clear`, `--continue` resumes the pre-clear conversation; `--adopt` with the id `lyx orch status` shows still works.
- After a `/clear`, the watcher writes the new id up to one poll interval later, so a power loss in that window leaves the pre-clear id.
- A strand that reed's rebirth relaunched replays `--resume <id>`, so the watcher records the pre-clear id there.
- `--continue` does not check the resumed session's size; the watcher cycles it as any session.
- The shuttle session-reading change touches `internal/shuttleengine`, which `creel-stage1-fixes` also changes; it only adds a field to the result, so a merge conflict there should be trivial.
- Existing `TestReadSessionStates` rows with no record id now assert `recorded-session` instead of an empty `SessionID`, because the `writeReadRun` helper defaults an empty record id to that value.
