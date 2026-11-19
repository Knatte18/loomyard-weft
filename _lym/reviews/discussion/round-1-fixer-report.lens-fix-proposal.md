```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out (fix-proposal lens).

1. MEDIUM — F11's new D6 criterion, "a regeneration whose hook returns a member named `lym.generated`, `folders` or `settings` returns an error and writes no workspace file".
   Summary: the current write order breaks the criterion for a beam file's member.
   Evidence: in `regenerateHubWorkspace` (`internal/fabricengine/hubworkspace.go` ~178–183), the hub file is built and written through `writeHubWorkspace` before the beam loop (~192) builds each beam file.
   D6 refuses `lym.generated` as "the beam file's generated mark", so that refusal comes after the hub file is already written.
   The record never says the regeneration builds every file before it writes any.
   Without that rule, the criterion cannot pass.

2. MEDIUM — F3's new D5 text: "A server removes any leftover state file right after it takes the lock … so the state that answers is the live server's."
   Summary: the guarantee does not hold in the gap between the new server taking the lock and removing the old state file.
   Evidence: in that gap, `open` reads the leftover state and sends a request to the old port.
   The review's own F3 premise says "That port may now belong to another process."
   "Answers a request carrying its token" asks for no proof that the responder knows the token, so any listener that answers passes.
   `open` then hands the browser a launcher for a foreign port.

3. MEDIUM — F10's new D5 bound: "at the cap the verb leaves a still-running opener to finish on its own and returns success".
   Summary: this contradicts a sibling text and leaves a child process unhandled.
   Evidence: the record's Constraints line for `PATTERN-child-processes` still says "the opener is waited on".
   The capped opener becomes an orphan: it is not reaped through `proc.Reap` and not registered in `runproc`.
   The record states no bound for that departure from PATTERN-child-processes.

4. LOW — F4's new D5 and D11 text: the server always starts in the prime's anchored directory.
   Summary: there are two gaps, and the record states neither.
   - In a beam window, page writes use the prime's config, while the CLI from that window uses the beam's.
     `ConfigDirs(layout)` builds its `Anchor` layer from `l.AnchorPath()` (`internal/fabricengine/junctionnames.go:130-135`), so this conflicts with D10's bound that the page "checks the CLI's verb runs".
   - D6's bound already allows a window where "the prime did not resolve". There, `open` cannot start a server, and no refusal or way forward is named for that case.

5. LOW — F3's new D11 exit order: "releases its lock … wakes every waiting `GET /api/board` at once with 'unchanged'; it then stops taking requests".
   Summary: a woken page asks again at once, and that new long poll can land before the server stops taking requests.
   The exit then waits up to the long-poll bound, which is under a minute. Stopping intake before waking the polls avoids this.

6. LOW — F7's new D10 release text: "(`running · &lt;producer&gt;`, as `boardengine.IsRunStatus` reads it)".
   Summary: the parenthetical shows only the `running` form, but the rule covers more.
   Evidence: `internal/boardengine/runlock.go:18-28` — `IsRunStatus` accepts any `&lt;state&gt; · &lt;producer&gt;`, and `RunStatus(state, producer)` puts the run's FSM state in the first half. So release is refused for any run state, not only `running`.

Tool calls:
- Read `runlock.go` and `ConfigDirs`: to check what `IsRunStatus` accepts and which config layer the anchor feeds. My context lacked both.
- Grepped `hubworkspace.go` for the task cwd and `RegenerateHubWorkspace`: to check the write order. My context lacked it.
- Read `regenerateHubWorkspace` lines 139–240: to confirm the hub file is written before the beam files are built.
