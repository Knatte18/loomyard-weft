# Decision record — creel CLI output fixes

## Goal

Fix three output bugs in `lyx creel` found in the creel stage 1 re-test (#699, #700, #701):
a success envelope that cannot be written is lost with exit 0,
the stderr fallback of a refusal prints `creel: ` twice,
and `log --id <id>` leaves out the `reminded` events that list that id.

## Scope

In:

- `internal/creelcli`: the shared success path of `runVerb`, the stderr fallback in `writeRefusal`, the post and inject bodies naming what they already stored, and the package `doc.go`.
- `internal/creelengine`: the `--id` match in `Store.Log`, and the `# Log` section of the package `doc.go`.
- `tools/sandbox/SANDBOX-CREEL-SUITE.md`: its `log --id` expectation, which lists only `posted` and `read`, gains the `reminded` events D4 adds.

Out:

- The hidden `daemon` verb's two `output.Ok` lines: the daemon spawn gives it no stdout a caller reads.
- `internal/output`: `output.Ok` keeps its signature and its unchecked write; creelcli stops calling it on the verb success path instead of changing a helper other modules share.
- The engine's error texts: every `creel: ` prefix in `internal/creelengine` stays.
- The inject verb's stall notes on stderr, which are progress lines, not the answer.
- The id of a waiting post or inject that times out: `Client.Call` answers `ErrWaitTimeout` with an empty response, so the timeout refusal names no id even on a writable stdout. That gap is in the client, not in the output path, and is listed under Open risks.

## Decisions

### D1 — A success envelope that cannot be written exits 1 with one stderr line

`runVerb` writes a zero-code result's fields through creelcli's own `writeOk`, which returns the write error, instead of `output.Ok`.
When that write fails, the verb records exit 1 and writes one line to the command's stderr: `creel: <done>, but its answer could not be written: <write error>`.
`<done>` is the verb's own statement of what it already did, when it stored something (D2);
a verb that stored nothing writes `creel: the answer could not be written: <write error>`.
This covers every verb that answers through `runVerb`: post, inject, interrupt, forward, inbox, log, hold and release.

Rationale: the cause named in #699 is the one unchecked write on the shared path;
`writeOk` already exists for `read` and already returns the error, so every verb gets the same check at one site.

Bound: only the write of the success envelope is newly checked; the answer itself, and any state the daemon already changed, are untouched.
A write that fails after part of the line went out still exits 1 and still writes the stderr line, so a consumer may hold a truncated line and a non-zero exit.
The stderr write's own error is ignored, as `writeRefusal` does today, so with both streams refusing the exit is still 1.

### D2 — post and inject name the id they stored

`result` gains a field that names what the verb already did, set by the bodies that store something before the answer is written:

- `post` to a receiver and `post --reply`: `message #<id> was posted`.
- `inject`: `inject #<id> was queued`.

D1's stderr line opens with it.
`writeRefusal`'s stderr fallback opens with it too, for the non-zero results these verbs answer after storing (inject `--wait` unconfirmed or expired, post `--wait-reply` returned undelivered), as `creel: <done>; <msg>`.
A refusal built from an error, the wait timeout included, carries no `<done>`, since the body holds no id there (see Scope).

Rationale: the board entry asks the stderr line of a post or inject to name the id, since the message or inject was already stored.

### D3 — The stderr fallback prints the `creel: ` prefix once

`writeRefusal` prepends `creel: ` only when the message does not already open with `creel: `.
A message opening with another form, such as `creel read: ...` or `creel needs a hub ...`, still gets the prefix, so every fallback line opens with `creel: `.
When D2's `<done>` is present the line opens with `creel: <done>; ` and `<msg>` follows unchanged.

Rationale: engine errors open with `creel: ` and creelcli's own refusals mostly do not;
checking the prefix at the one site fixes both without rewording any error.
`creel: creel read: ...` stays as it is: the board entry calls it intended, and it keeps the `creel: ` opening every fallback line shares.

### D4 — `log --id` matches a reminded event that lists the id

`Store.Log`'s id filter keeps an event whose `ID` equals the filter, or whose `Unread` holds it.
The other filters (`From`, `To`, `Since`) apply to the event as before, so `--id 2 --to lyt:eve` keeps the reminders to `lyt:eve` that list #2.
A reminded event carries no `From`, so `--id` with `--from` drops it, as `--from` alone does today.

Rationale: since #664 a `reminded` record carries its ids in `Unread` and no `ID`;
matching `Unread` is event-agnostic and covers any later record that lists ids the same way.
Mail and injects take ids from one counter, so an inject's `--id` view never picks up a reminder about a different message.

### D5 — Docs land in the same commit

- `internal/creelcli/doc.go`: a success envelope that cannot be written exits 1 with one `creel: ` line on stderr naming what post and inject already stored, and the fallback line carries one prefix.
- `internal/creelengine/doc.go` `# Log`: `--id` also lists the `reminded` events whose unread ids hold that id.

## Constraints

- `PATTERN-creel-store`: the change to `Store.Log` stays inside `internal/creelengine`, and creelcli still reaches the journal only through the client.
- `PATTERN-test-economy` and `PATTERN-test-speed`: D4's scenarios are pinned untagged in creelengine's store tests with the fake delivery; D1–D3 extend the integration-tier failing-writer subtests in creelcli's `TestCreelCLI`, so the plan's verify runs `--tags integration` on `internal/creelcli`.
- `PATTERN-tolerant-records`: no record shape changes.
- Test budget: extending subtests needs no `cmd/lyx/testdata/test-budget.yaml` edit; a new top-level test past a package's row lists the file under `Edit:`.

## Auto-mode assumptions

- Exit 1 for a lost success envelope, not a new exit constant: the board entry says "as for a refusal".
- The `<done>` wording (`message #<id> was posted`, `inject #<id> was queued`) is this record's pick; the plan may reword it as long as the line names the id and opens with `creel: `.
- The fallback of non-zero post and inject results names the id too (D2), beyond the board entry's success-only wording, since it is the same field and the same stored state.

## Open risks

- Existing creelengine tests compare the exact event list of `LogFilter{ID: ...}`; a test whose message was reminded now sees `reminded` events in that list.
  The plan checks each such assertion and updates the ones D4 changes.
- A `post --wait-reply` from an EXTERNAL caller whose reply arrived but whose envelope cannot be written names only the posted id; the reply was handed to the waiter and marked delivered, so it is not in any mailbox and only `lyx creel log` shows it.
- A waiting post or inject that times out has stored its message or inject, but the refusal names no id (see Scope); the caller finds it with `lyx creel log --since`.
- Until `stencilstore-dev-warning-noise` lands, a dev build prints a stencilstore warning on stderr for every creel verb, so a sandbox re-test sees an extra stderr line; the acceptance criteria are checked against the command's own stderr writer.

## Acceptance criteria

- Each of post, inject, log and inbox (from a strand), run into a writer that refuses writes, exits 1 and writes one stderr line opening with `creel: `.
- That line names `#<id>` for post, post `--reply` and inject, and the message or inject is stored all the same.
- A verb whose success envelope is written still exits 0 and writes nothing to the command's stderr.
- The existing fallback cases keep their exit codes: inject `--wait` unconfirmed exits 2, expired 3, a wait timeout 4, and post `--wait-reply` returned undelivered 5, each with one `creel: ` line; the unconfirmed, expired and returned lines name the id.
- Into a refusing writer, an engine refusal (unknown receiver, or an EXTERNAL caller's inbox) and a creelcli refusal (a standalone directory, or a post with the wrong argument count) each print exactly one `creel: ` prefix.
- `read` into a refusing writer still exits non-zero and leaves its mail unread.
- `log --id <id>` of a message that was reminded lists `posted`, each `reminded` event whose unread ids hold it, and `read`, oldest first.
- `log --id <id>` leaves out a `reminded` event whose unread ids do not hold it.
