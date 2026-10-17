# Fix creel CLI output: lost success, doubled prefix, log --id

Fixes three output bugs in `lyx creel` found in the creel stage 1 re-test.

- A success envelope that cannot be written (stdout is `/dev/full`, say) no longer exits 0 with an empty stderr.
  The verb exits 1 and writes one line to its own stderr: `creel: <done>, but its answer could not be written: <write error>`.
  `<done>` names what the verb already did: `message #<id> was posted` for post and `post --reply`, `inject #<id> was queued` for inject, and `the <verb> took effect` for forward, hold, release and an interrupt that stopped a turn.
  An interrupt that found no running turn, inbox and log change nothing, so they write `creel: the answer could not be written: <write error>`.
  `runVerb` now writes through creelcli's own `writeOk`, which returns the error, instead of `output.Ok`.
- The stderr fallback of a refusal prints `creel: ` once.
  `writeRefusal` adds the prefix only when the message does not already open with it, so `creel: creel: unknown receiver ...` becomes `creel: unknown receiver ...`.
  The non-zero results that post and inject answer after storing (inject `--wait` unconfirmed or expired, post `--wait-reply` undelivered) open with `creel: <done>; <msg>`.
- `lyx creel log --id <id>` also lists the `reminded` events whose `Unread` holds the id, so one message's history (posted, reminded, read) reads in one call.
  `--from` with `--id` still drops reminders, since they carry no sender.

`tools/sandbox/SANDBOX-CREEL-SUITE.md` gains the `reminded` events in its `log --id` expectation.
The `doc.go` of `internal/creelcli` and `internal/creelengine` describe the new behaviour.

Closes #699
Closes #700
Closes #701

## Design decisions

- Exit 1 for a lost success envelope, not a new exit constant, matching a refusal.
- Only the success envelope write is newly checked.
  The exit 1 does not undo what the daemon already did, and a write that fails midway can leave a truncated line with a non-zero exit.
- `output.Ok` is unchanged, since other modules share it; the hidden `daemon` verb keeps its unchecked write.
- The `Store.Log` match is on `Unread` rather than on the `reminded` event name, so any later record that lists ids the same way is covered.

## To check

- The id gap: a post or inject refused from an error after the daemon stored it (wait timeout, or a connection lost after the store) still names no id, because `Client.CallWatching` returns an empty response there.
  The caller finds it with `lyx creel log --since`.
- A `post --wait-reply` from an EXTERNAL caller whose reply arrived but whose envelope cannot be written names only the posted id; the reply is marked delivered and shows only in `lyx creel log`.
- Refusals cobra raises before a verb body runs (argument count, missing flag, unknown subcommand) are written by `clihelp` and keep their current behaviour, so they write no stderr line on an unwritable stdout.
- A dev build prints a stencilstore warning on stderr for every creel verb until `stencilstore-dev-warning-noise` lands, so a sandbox re-test sees an extra stderr line.
- The `SANDBOX-CREEL-SUITE.md` sandbox suite was edited but not re-run by hand.
