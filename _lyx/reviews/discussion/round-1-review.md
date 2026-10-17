---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, Acceptance criteria, bullets 3 and 4
    summary: The criterion that a written success envelope leaves the command's stderr empty, and that each fallback case writes one creel line, is false for a waiting inject that stalls, whose stall notes Scope keeps on stderr.
    fix: >-
      In decision-record.md Acceptance criteria, reword bullet 3 to "A verb whose success envelope is written still exits 0 and writes no `creel: ` answer line to the command's stderr; a waiting inject's stall notes are progress lines and may precede it." and in bullet 4 change "each with one `creel: ` line" to "each with one `creel: ` answer line after any stall notes".
    origin: lens:bound-breaker
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, Scope (last Out bullet) and Open risks (third bullet)
    summary: The record names the wait timeout as the only path where a stored post or inject is refused with no id, but a connection lost after the daemon stored it (a plain post's lost answer, or a waiting call that gives up its reconnects or fails to resume) is the same gap.
    fix: In decision-record.md, widen the last Scope Out bullet and the third Open risks bullet from the wait timeout alone to every refusal the client builds from an error after the daemon stored the post or inject, naming the timeout and a connection lost after the store, since `Client.CallWatching` answers both with an empty response; keep the way forward, `lyx creel log --since`.
    origin: lens:bound-breaker
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, D1
    summary: D1 does not say whether its stderr line is written directly or through writeRefusal; through writeRefusal the verb would first write a JSON error envelope to the same stdout, a second line after a partial success write, and get D3's `<done>; <msg>` shape instead of D1's.
    fix: In decision-record.md D1, add after the stderr-line sentence one sentence that the line goes straight to the command's stderr, never through `writeRefusal`, so no error envelope follows the failed success write on stdout.
    origin: lens:interactions
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, D1, third sentence
    summary: D1 calls interrupt, forward, hold and release verbs that stored nothing, yet each journals a change before its answer, so the label contradicts D1's own bound that state the daemon already changed is untouched.
    fix: In decision-record.md D1, replace "a verb that stored nothing writes" with "a verb that names no stored id (interrupt, forward, inbox, log, hold, release) writes", and add to the Bound paragraph that for interrupt, forward, hold and release the exit 1 does not undo what the daemon already did.
    origin: lens:interactions
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md, Scope Out and D5
    summary: Cobra-level refusals (a wrong argument count under ExactArgs, a missing required --for, a bad flag, an unknown subcommand) go to output.Err on stdout in clihelp.RunRootCtx and never reach writeRefusal, so on an unwritable stdout they stay lost with no stderr line, and the record and D5's doc line are silent about it.
    fix: In decision-record.md Scope Out, add a bullet that refusals cobra raises before a verb body runs are written by `clihelp` and keep their current behaviour; in D5's `internal/creelcli/doc.go` bullet, limit the fallback claim to refusals the verbs write.
    origin: lens:existing-paths
---

### [LOW] An acceptance criterion contradicts the stall notes Scope keeps

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria, bullets 3 and 4.

**Issue:** The `inject` progress callback writes `creel: <receiver> is stalled: ...` to `cmd.ErrOrStderr()` (`internal/creelcli/cli.go:338-341`), and `TestCreelCLI` pins those lines on stderr (`internal/creelcli/cli_test.go:411-466`).
An `inject --wait` that stalls and then lands exits 0 with its envelope written and stall lines already on stderr.
That falsifies "writes nothing to the command's stderr".
A timed-out inject that stalled first likewise holds more than "one `creel: ` line".
The interactions lens raised the same defect, and this entry is its deduplicated form.

### [LOW] The stored-but-unnamed case is wider than the wait timeout

**Location:** `_lyx/discussion/decision-record.md`, Scope (last Out bullet) and Open risks (third bullet).

**Issue:** `Client.CallWatching` returns `Response{}` with an error in three more stored-but-unnamed cases.
A non-waiting call whose answer is lost after the daemon stored the post returns at once (`internal/creelengine/client.go:155-157`).
A waiting call that has already read the `Pending` id into `state.id` and then gives up its reconnects returns the same way (`client.go:158-160`).
So does a waiting call that then fails to resume (`client.go:161-162`).
D2's rule that an error-built refusal carries no `<done>` still holds.
But Scope and Open risks present the timeout as the only case, so the plan and the `log --since` guidance miss the others.
The existing-paths lens raised the same gap, and this entry is its deduplicated form.

### [LOW] D1 leaves open how its stderr line is written

**Location:** `_lyx/discussion/decision-record.md`, D1.

**Issue:** `writeRefusal` writes a JSON error envelope to stdout first and falls back to stderr only on failure (`internal/creelcli/cli.go:103-117`).
D1 never says whether its line uses that path.
The only hint is the bound's "as `writeRefusal` does today".
A plan that routes D1 through `writeRefusal` writes a second line to stdout after a partial success write.
It also yields `creel: <done>; <msg>` rather than D1's `creel: <done>, but its answer could not be written: <err>`.

### [LOW] D1 mislabels state-changing verbs as storing nothing

**Location:** `_lyx/discussion/decision-record.md`, D1, third sentence.

**Issue:** Forward appends a `forwarded` record and delivers (`internal/creelengine/verbs.go:305-308`).
Hold, release and interrupt journal `held`, `released` and `interrupted` (`internal/creelengine/journal.go` event constants).
On a refusing stdout these verbs exit 1 with a line saying only that the answer could not be written.
Calling them verbs that "stored nothing" contradicts D1's own bound that state the daemon already changed is untouched.
It also leaves the reader of an exit 1 unaware that the action took effect.

### [LOW] Cobra-level refusals bypass the fallback the record and D5 describe

**Location:** `_lyx/discussion/decision-record.md`, Scope Out and D5.

**Issue:** `clihelp.RunRootCtx` answers a cobra error with `output.Err(out, ...)` and records exit 1 (`internal/clihelp/exec.go:180-185`).
Those errors include `cobra.ExactArgs` on inject, interrupt, forward, hold and release, and `MarkFlagRequired("for")` on hold.
None of them pass through `writeRefusal`, so on an unwritable stdout they are lost with no stderr line.
D3's own claim, that every fallback line opens with `creel: `, holds.
But D5's doc line, and the existing `doc.go:11` sentence it rewrites, read as covering every refusal.

## Rejected

- Walkthrough lens: no findings; its cases (post and inbox into a failing writer, inject unconfirmed, post returned, the timeout, the engine and creelcli refusals, and `log --id` over reminders) agree with the code the handler read.
- D1 and D3 using different joiners (`, but its answer could not be written:` against `; `): the two lines describe different things, a lost success against a stored-then-refused result, and D3's prefix rule governs only `writeRefusal`'s fallback; the routing ambiguity that makes the difference matter is kept as F3.
- Handler check of D4's premise: held. `journalReminded` is the only writer of `Unread` (`internal/creelengine/deliver.go:555`), a `reminded` entry carries `ID` 0 and no `From` (`fold.go:383-395`), `high-water` records never reach the event list (`fold.go:228-229`), and both id issue sites use `newID` (`verbs.go:32`, `verbs.go:379`), so matching `Unread` on every event cannot pull in an unrelated event.
- Handler check of D2 against exit codes 2 to 5: held. The unconfirmed, expired and returned results carry `resp.ID` in their fields (`cli.go:290-301`, `cli.go:352-363`), and the timeout path hands `writeRefusal` only `res.fields` with no id (`cli.go:161-163`).
- Handler check of acceptance-criteria coverage: D1 (bullet 1), D2 (bullets 2 and 4), D3 (bullet 5) and D4 (bullets 7 and 8) each have a must-cover scenario; D5 is docs only.
- Handler check of the completeness-before-leanness test: every requirement in Open risks and Auto-mode assumptions lives in `decision-record.md` itself; `support-log.md` holds only deliberation and rejected alternatives, so nothing needs extracting.
- Handler check of the attack-surface and board-lock rules: no new verb, flag or escape hatch is introduced, D1 carries its bound, and no Decision edits the task's own board entry.
