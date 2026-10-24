# Restore to PATTERN the moved rules that bind callers outside their package

`pattern-restore-caller-rules` · Task · audit, config, shed, reed, fabric · running · PR-Review

## Seen (audit of 04aafa88c, 2026-10-24)

`pattern-cleanup` moved 21 package-local rules out of `PATTERN.md` into their owning package's `doc.go`.
A rule an agent in another package can break, with no test to catch it, is no longer in front of that agent.
The audit found no dangling `PATTERN-` ID and no clause dropped outright.

## Back to PATTERN (binds outside code, no scan catches a break)

- `sparse-hub-base` (now `internal/hubreconcile/doc.go`): every writer of a hub config file, by any package, must keep it sparse.
- `parent-lock` (now `internal/parentlock/doc.go`): any package that moves or lands on a local parent branch can break it.
- `agent-name` (now `internal/agentname`): anything can hand-form a strand name; `spawnrole_test` covers only role literals.
- `wait-mark-display-only` (now `internal/reedengine/doc.go`): any package could read `@lyx_wait` and decide on it.

## Unsure, leaning back

- `transient-stop` (now `internal/shedengine/doc.go`): no scan of `MarkTransient` callers.
- `friction-capture` (now `internal/friction/doc.go`): a new refusal or halt path outside the package must write a note; review covers it.
  Its clause that `lyx loom start` vouches for the resume it spawns now lives only in a comment at `internal/loomcli/start.go`.

## Unsure, leaning fine

`hub-wide-config`, `recipe-owns-failures`, `batcher-registry`.

## Idea

Restore the four, and decide on the two leaning back, as short overview lines, keeping each rule's detail in its `doc.go`.
A tripwire test for a restored rule would let it stay out of the overview later.
