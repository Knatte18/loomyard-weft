```yaml
lens: row-validity
outcome: reported
```

Lens row-validity: whether each planned test row reaches its branch, would fail if the code were wrong, and is not redundant. The defects found:

1. **MEDIUM — card 9 (`TestRunRelay`): a row waits out the 10-second re-exec lock wait in real time, and no row tells a fresh start from a re-exec.**
   - The re-exec branch fires when `RelayState.PID` equals the relay's own pid. Every `RunRelay` in a test runs in the test process, so a second run in the same geometry sees its own pid and takes the re-exec branch.
   - Row "a second `RunRelay` while the first holds the lock returns before any read" therefore waits the full 10 seconds on the lock before it returns. That breaks the `PATTERN-test-speed` rule that no test waits out a production duration ("Time is injectable").
   - Row "the lock is free for a later run, which takes it" also takes the re-exec branch, so the fresh-start try-only branch has no row that would fail if it waited.
   - `RelayDeps` names no seam for the re-exec wait or for the pid the check compares, so a test cannot avoid either.
   - Evidence: the card 9 Intent, step 2 ("a relay whose `RelayState.PID` is its own pid is a re-exec and waits up to 10 seconds") and the `RelayDeps` field list.

2. **MEDIUM — card 9 (`TestEnsureRelay`): the planned rows cannot reach the spawn the card says tests replace.**
   - `EnsureRelay(geom Geometry)` and `EnsureRelayWhileAway(geom Geometry)` take no seam, and `starter` is unexported with no stated way to inject it.
   - Rows "a down relay answers `started` with one spawn" and "`EnsureRelayWhileAway` spawns … while presence is `slack`" need a replaced spawn. Under `go test` the production spawn returns `ErrSpawnUnderTest`.
   - The card does not say whether the replacement is a package-level var, which would rule out `t.Parallel` and needs naming, or a function over `starter` that the rows call.
   - Evidence: card 9's `Create` heads for `EnsureRelay`, `EnsureRelayWhileAway` and `plan:internal/slackengine#starter`.

3. **MEDIUM — card 10 (`TestSlackCLI`): a tier0 test that drives `Command()` "without a hub" has no declared entry point.**
   - `Command()` takes no arguments. A `lym` command resolves its location from the cwd, which needs a hub, and that makes it a tier1 hubforge test.
   - The rows need a hand-built `slackCLI` with fake seams, but the card declares no constructor or method that builds the tree over one (for example `func (c *slackCLI) command() *cobra.Command`).
   - It also does not say what a nil seam means on a hand-built receiver: nil ensure, nil stop, nil await, nil clock, nil beams or nil pause reader.
   - Evidence: card 10's `Create` list and its Intent ("`slackCLI` holds the location, the `slackengine.Geometry` and seams … so tier0 tests drive every verb without a hub").

4. **MEDIUM — card 11 (`TestResumeContext`): the case where `ResumeContext` returns the pending pointer verbatim has no row, and as worded it can deliver the pointer twice.**
   - `Watcher.startClearing` stores `st.PendingResume = resume` (`internal/orchengine/watcher.go:1207`). Card 11 passes that pointer through `WithPresence`, so the stored text already carries the presence line.
   - `ResumeContext`'s verbatim branch returns `st.PendingResume`. If it also applies `WithPresence`, which "every case" in the card's wording suggests, the mark text gets the line twice.
   - `Watcher.hookDelivered` skips typing only when `mark.Text == st.PendingResume` (`internal/orchengine/watcher.go:1365`). A doubled line breaks that equality, so the pointer is typed again after the hook already delivered it.
   - None of the planned rows covers PhaseResuming with a pending pointer and presence `slack`.
   - Evidence: card 11's presence-line bullet; `internal/orchengine/resumecontext.go:25-38`.

5. **MEDIUM — card 11 (`TestStart_StartsTheDobbiesFirst`): the row "a nil relay seam, a beam's wiring, calls nothing" does not reach `wireCLI`'s prime-only fill.**
   - The harness sets `h.cli` fields by hand (`internal/orchcli/startdobbies_test.go:60`) and never runs `wireCLI`.
   - So the branch that fills `startRelay` only for the prime, and leaves it nil for a beam, goes untested. A `wireCLI` that wired the relay for beams would still pass.
   - Evidence: card 11 lists `internal/orchcli#wireCLI` under Edit, but no row reaches it.

6. **MEDIUM — card 13 (`TestChangeAnchor`): two branches next to the new relay restart have no row.**
   - The card's own rule says a prime move that fails before `EndDobbies` starts no relay. The fake can produce that state through `lockErr` (`internal/fabriccli/anchor_test.go:53`, `:200`), but no row asserts that the log holds no `start-relay`.
   - A prime move whose `EndDobbies` itself fails (`endErr`, with `endedBeforeErr`, `internal/fabriccli/anchor_test.go:54-56`) has no row either, although the relay may already be ended by then.
   - Evidence: card 13's Intent bullets 2 and 4.

7. **LOW — card 3 (`TestResolveSender`): the planned new row repeats an existing one.**
   - `Linux_ProgramClaimFromANonOrchPaneIsRefused` already reaches this exact refusal (`internal/creelengine/identity_test.go:41`, which checks `wantErr` "only an orch strand's pane").
   - The new behavior is the sentinel, so this should be an `errors.Is(err, ErrProgramClaimInStrandPane)` assertion on that existing row, not a new row.

8. **LOW — card 10 (`TestPaneRefusal`): a separate top-level test of the unexported `paneRefusal` mostly repeats `TestSlackCLI` rows, and the off-Linux branch has no row.**
   - `TestSlackCLI` already reaches the orch-allowed, orch-refused, non-orch-pane and unknown-pane outcomes through the verbs (the public surface).
   - Only the "unreadable strands" row is new, and it fits as a `TestSlackCLI` row.
   - The off-Linux `LYM_STRAND_NAME` branch, which `paneCaller.GOOS` and `StrandName` select, is reached by no row in either test.

9. **LOW — cards 7, 8 and 10: a few rows repeat what another test covers.**
   - Card 10's `away` row asserting "mails each orch one notice" repeats `TestSetPresence`'s change row (card 7). `away` should assert only that it calls the presence change after the await.
   - Card 8's `TestOutboundText` rows "a non-orch strand and EXTERNAL answer false" repeat `TestOutboundRead`'s "not posted" rows for the same senders.

10. **LOW — card 9: `LastSlackOK`, which `lym slack status` prints, is written by no planned row.**
    - The `TestRunRelay` rows assert the Slack call order and the lock, but none asserts that a successful call writes `LastSlackOK`. A relay that never wrote it would pass.

Tool calls made:
- grep for the named test functions, the watcher members, the tier of `hubgeom_test.go` and `newRig`'s daemon options — to confirm that the glyphs cards 2, 3, 4 and 11 name exist, and how the creel rig resolves panes.
- read of `internal/orchengine/watcher_test.go:1496-1560` and the `PendingResume` and mark-compare lines in `internal/orchengine/watcher.go` — to check card 11's watcher row and the verbatim path.
- read of `anchorFake`'s fields and the `deps.Prime` uses in `internal/fabriccli/anchor_test.go` and `internal/fabriccli/anchor.go` — to check which failure states card 13's rows can script.
- grep of `daemon.fail` and the rows of `TestResolveSender` — to check card 3's planned row against the existing ones.
