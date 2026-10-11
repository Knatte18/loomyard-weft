---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D5 (step 2 and rationale) and the third acceptance criterion
    summary: A file setting exit-unattached on ends the server as soon as the claim's client disconnects, so D5's post-claim pin in a separate invocation cannot save it and the criterion expecting the boot to succeed cannot pass
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5 step 1 and D8 (tag set beside the session-local pin)
    summary: A concurrent boot's session is unguarded and untagged between its spawn and its own pins, so a winning claim's global destroy-unattached kills it and a concurrent teardown can kill the server under it
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3 step 1
    summary: The pre-spawn liveness rule makes a boot a join when the server it saw dies before its spawn, so its spawn starts an unclaimed server that never sources the file
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D2 and D3 step 2
    summary: The conditional claim nests source-file and the path inside a tmux command string, and D2 refuses glob characters but not whitespace or tmux quoting characters, so a path with a space is split and never loads
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D8 (discovery, reaping and status sentence)
    summary: D8 names surfaces instead of a rule, leaving the next and previous session switch and the status bar session loop showing and entering file-created sessions
---

### [BLOCKING] `exit-unattached on` kills the server before D5's post-claim pin runs

**Location:** `_lyx/discussion/decision-record.md` D5 step 2 and the rationale sentence "A file setting `exit-unattached on` leaves the session up through the source on tmux 3.6, so the post-claim pin is enough for it"; the third acceptance criterion.

**Issue:** The rationale is false on tmux 3.6.
tmux's server loop exits when `exit-unattached` is on and no client is connected.
Reed runs each tmux command as its own client process, so the moment the claim's client disconnects the server has no clients and exits, before step 2's pin can connect.
Probed on a scratch tmux 3.6 server (`-f /dev/null`, one detached session, file `set -g exit-unattached on`):

- `source-file` then a separate `set -s exit-unattached off`: five runs out of five, the pin got "server exited unexpectedly" and `has-session` found no server.
- `source-file f \; set -s exit-unattached off` in one invocation: the session survived.

So under the stated ordering, the third acceptance criterion's file, which sets `set -g exit-unattached on`, ends the server, step 3 finds the session gone, and the boot fails with an error that blames `destroy-unattached` or `kill-session`.
That criterion cannot pass.

**Fix:** Decide that the post-claim pins (`-g destroy-unattached off` and server `exit-unattached off`) are part of the same tmux invocation as the claim, after the conditional source, so one client stays connected from source through pin.
Every boot sends that invocation, so joins and losing candidates get the pins too.
Correct the rationale to say why the pins must share the claim's invocation, and keep step 3 as a separate check after it.

### [MEDIUM] A concurrent boot's session is unguarded between its spawn and its own pins

**Location:** `_lyx/discussion/decision-record.md` D5 step 1 ("Before the claim of D3, reed sets `destroy-unattached off` as a session-local option on the boot's own session") and D8 ("set at boot beside D5's session-local pin").

**Issue:** Step 1 and D8's tag are separate tmux invocations after the spawn.
Concurrent boots are not serialized (D3 says so), so boot B's `new-session` can land between boot A's winning claim and A's `-g` pin.
Probed on tmux 3.6: with the file's `set -g destroy-unattached on` applied, a session created by a later `new-session -d` died at once (`can't find session`), while one created by `new-session -d … \; set destroy-unattached off` in one invocation survived.
B's session dies before B's step 1 runs, and B's step 3 then fails B with a message telling the operator to remove a command from the file, though B's own boot did nothing wrong and a retry succeeds.

D8 has the same window: on a `sourced` server, B's session before its tag does not count as reed's, so a concurrent last-worktree `down` counts zero reed sessions and kills the server under B's half-finished boot.
Before D8 that `down` would have seen B's session in `list-sessions` and left the server up, so D8 introduces this race.

The third open risk covers an untargeted command hitting another worktree's session, not either of these windows, and no acceptance criterion exercises a concurrent boot against a file that sets `destroy-unattached`.

**Fix:** Decide that the session-local `destroy-unattached off` and the `@lyx_reed_session` tag are chained into the spawn invocation itself (`new-session … \; set …`), so they apply in the same command queue that creates the session and no other client's commands can run between them.
State in D3 that the spawn argv gains these chained commands while keeping `-f /dev/null`.
Add an acceptance criterion: two worktrees booting concurrently into a fresh hub with a file that sets `set -g destroy-unattached on` both succeed.

### [MEDIUM] The candidate rule misses a server that dies between the pre-spawn check and the spawn

**Location:** `_lyx/discussion/decision-record.md` D3 step 1 ("A boot is a candidate only when, before its spawn, the hub socket answered with no live server. A boot that found a live server is a join and sources nothing.").

**Issue:** `Down` and `EndSessionByName` kill the server when no session remains, and reed's op lock is per worktree, so they are not serialized with another worktree's boot.
When boot B checks the socket while worktree A's last session is still up, B is classed as a join.
If A's `down` then kills the server, B's `serverSpawnArgv` starts a fresh server.
That server carries no claim, B sources nothing, and every later boot finds a live server and joins.
The file is then missing until the server's next start, contradicting D3's rule that a value takes effect at the next start, and D8's tag rule is off on that server because its claim is not `sourced`.
No acceptance criterion covers this, and the open risks do not name it.

**Fix:** Decide a starter test that reads the server the spawn produced rather than one observed before it.
For example, record the server's `#{pid}` before the spawn (none when nothing answers), and make the boot a candidate when the server answering after the spawn has a different pid.
A pre-change server keeps its pid across a join, so it is still never a candidate.
Alternatively, accept the race as an open risk and state its bound: which servers can run without the file, and what clears it.

### [LOW] The claim's nested command string is not covered by D2's path rules

**Location:** `_lyx/discussion/decision-record.md` D2 (refused characters `*`, `?`, `[`, `\`) and D3 step 2 ("one tmux invocation that, if … unset, sets it … and then runs `source-file <path>`").

**Issue:** A conditional set-and-source in one invocation needs `if-shell -F` (or an equivalent), whose branch is a tmux command string that tmux parses again.
D2's rule protects the path from `source-file`'s glob expansion, not from that second parse.
Probed on tmux 3.6: `if-shell -F '#{@claim}' '' "set -s @claim sourced ; source-file <dir>/sp ace/c.conf"` reported "No such file or directory" for `<dir>/sp` and `ace/c.conf` and loaded nothing.
D4 would reduce that to a warning, the silent non-load D2's rationale sets out to prevent.
Spaces, quotes, `;`, `#` and braces all hit the same parse.

**Fix:** Decide one: D2 also refuses whitespace and the tmux command-parser metacharacters, or D3 states that the path is quoted for tmux's command parser inside the claim.
Add a row to the path acceptance criterion, a path with a space either refused or loaded.

### [LOW] D8 lists surfaces instead of stating a rule

**Location:** `_lyx/discussion/decision-record.md` D8 ("Reed's discovery, reaping and status never attach to, reap or report a session that does not count as reed's").

**Issue:** D8 names three surfaces, so a reed surface it does not name keeps today's every-session behavior.
Two such surfaces show a file-created session to the operator: the next and previous session switch (`SwitchClient` cycles every `list-sessions` id) and the status bar's session row (the `#{S:}` loop in `statusFormatSessions`).
The operator sees a button for the file's session and can switch into it, which D8's intent of treating it as foreign does not decide either way.
This is a gap in the method, a list where a rule is needed, not a request to enumerate call sites.

**Fix:** State D8 as a rule over every reed read that lists or cycles the server's sessions, so all of them count only reed's sessions on a `sourced` server.
Alternatively, name the bar and the switch as deliberately showing every session, and give that choice its bound.

## Verdicts on the focus questions

- D2 and D3, which boot sources the file: decided, with the step-1 and step-2 rule, concurrent boots covered by the in-server claim, and the fifth and sixth acceptance criteria.
  The rule misclassifies one case (F3), and the claim's command string needs a path rule (F4).
- D5, pin order and scope: the `destroy-unattached` order holds for the booting session, and the `-g` pin covers sessions that boot after it.
  The `exit-unattached` premise is false (F1), and concurrent sessions are exposed between spawn and pin (F2).
- D4, bad-file behavior: accurate, not a defect.
  Probed on tmux 3.6: a file with an unclosed brace exits 1 and applies none of its lines, and a file with an unknown option between two valid lines exits 1 and applies both valid lines, also when nested in the claim's conditional.
  The rationale and the seventh and eighth criteria distinguish the two cases.
- Sessions not created by reed: decided by D8, with its bound stated, and the next-start rule holds through teardown.
  F2 (the untagged window) and F5 (surfaces outside D8's list) remain.
