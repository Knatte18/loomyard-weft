# Creel stage 1 re-test fixes

`creel-stage1-fixes` · Task · bug, creel, shuttle, claude · running · Discussion-Write

Fix the eight creel defects found by the creel stage 1 re-test of 2026-10-16 (report: the creel audit of 2026-10-16).
Fix #677 first: it is the worst, since it leaves creel down until a journal line is removed by hand.
The reed defects from the same re-test, #679 and #680, are out of scope.

Operator decision 2026-10-16: the eight creel stage-1 re-test defects run as one low-budget task.

## #677: a torn last journal line that ends in a newline is skipped, not cut

Seen: with `{"event":"posted","at":"2026-10-16T08:45:1` plus a newline as the last journal line, the creel daemon logged `skipping a torn last line left by a crash ... line=101` and appended new records after it.
At the next start that line sat in the middle of the file and the daemon refused it: `line 101 is not a valid record (unexpected end of JSON input); way forward: repair or remove that line by hand`.
Creel was down until the line was removed by hand.

Expected: the skipped line is cut, as a torn tail without a newline already is (`cutting a torn last line ... bytes=42`), so a later start does not fail on it.

Sandbox scenario C10.

## #678: the llm smoke test never reads the booted session idle

Seen: `lyx gate test --tags llm ./internal/creelcli` fails with `smoke_llm_test.go:51: the adapter never read the booted session idle` after 60.7 s, the same message as before the #657 fix.
In live sandbox use the #657 fix holds: a strand added with plain `lyx reed add --cmd claude` (placeholder `Try "fix lint errors"`) was reminded 0.5 s after mail was posted.
The test prints no pane capture, so the cause is unknown; one candidate is the Claude folder-trust dialog in the test temp hub, which the sandbox also showed on first boot.

Expected: the smoke test passes, and on failure it prints the pane capture so the cause can be read.

## #681: `read` to an unwritable stdout fails silently

Seen: `lyx creel read > /dev/full 2>err` exits 1 and `err` is empty.
The mail correctly stays unread (the #660 fix holds).

Expected: a short error on stderr, since the JSON error envelope cannot be written to stdout either.

Sandbox scenario C4.

## #682: a stale stall is printed as current

Seen: the owner `inject --wait` into lyt:bob printed `lyt:bob is stalled: the input box holds a draft ... (at 10:39:55)` 37 s after the draft had been cleared, then injected at once.

Expected: no stall line once the input box is empty, or one clearly marked as past.

Sandbox scenario C8.

## #683: the verb waits about 23 s for a daemon that has already exited

Seen: against a malformed journal the creel daemon died at once (daemon log 10:46:01.395), but `lyx creel inbox` returned its error, which now quotes the daemon log as the #661 fix intends, only after about 23 s.

Expected: the verb reports the error as soon as the daemon it started has exited.

Sandbox scenario C10.

## #684: carried-forward messages are invisible to `log`

Seen: the journal holds `{"event":"carried","id":900,...}` and the receiver `inbox` counts message 900, but `lyx creel log --id 900` returns `[]`.

Expected: `log` shows a carried message, at least as a `carried` event.

Sandbox scenario C10.

## #685: the removed-holder sweep takes 80 s

Seen: strands frank and gus were removed at 08:48:03; their held messages went `returned`/`undelivered` at 08:49:23, 80 s later.
The outcomes themselves were correct.

Expected: per sandbox scenario C11, within about 60 s.

## #686: an expired inject names no reason, and a stalled inject overruns its TTL

Seen: `inject lyt:dave ... --ttl 10s --wait` into a bash strand ended `expired` with no `stall` field, while Claude-adapter panes now name theirs.
Separately, a 20 s TTL into a too-short pane expired at 23.3 s, while a never-idle pane's 30 s TTL expired on time at 31.4 s.

Expected: an expired inject names a reason, such as "not an agent pane", or the outcome is documented; a stalled inject expires at its TTL.
