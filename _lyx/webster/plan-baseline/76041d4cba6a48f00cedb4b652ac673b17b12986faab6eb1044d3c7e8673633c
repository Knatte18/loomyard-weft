# Card 6 — orch-notify-over-creel

**Create:**
- `internal/orchcli#PrimeOrchStrand`
- `internal/orchcli#notifyPoster`
- `internal/orchcli#notifyClients`
- `internal/orchcli#notifyPrime`
- `internal/battencli#battenProgram`
- `internal/loomcli#loomProgram`
- `internal/burlercli#burlerProgram`
- `internal/webstercli#websterProgram`
- `internal/hibernatecli#hibernateProgram`
- `internal/orchcli#TestNotifyPrime_SignsAndPosts`

**Edit:**
- `internal/orchcli#NotifyPrime` -> `func NotifyPrime(location *lyxcwd.Location, program, line string) (posted bool, err error)`
- `internal/orchcli#PrimePaths`
- `internal/orchcli#TestNotifyPrime_QueuesInThePrimeOnly`
- `internal/battencli#rowNotices`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenCLI.loadedTacklerDeps`
- `internal/battencli#TestWire_NotifyWritesNothingWithoutAnOrchStrand`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/loomcli#landingDeps`
- `internal/loomcli#TestLandingDeps_ConfigNoticeSeamsOverRealHub`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/webstercli#websterCLI.wireHub`
- `internal/hibernatecli#hubDeps`
- `internal/hibernatecli#Command`
- `internal/landingshed#Deps`
- `internal/sheddriver#handBackRun`
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/orchcli/doc.go`
- `internal/hibernatecli/doc.go`
- `contracts/stencils/orch/orch-template-role.md`
- `docs/overview.md`

**Delete:**
- `internal/orchcli#assertNotices`
- `internal/battencli#TestWire_NotifyQueuesOnThePrimesOrch`

**Intent:**
Every notice a module sends the orch is creel mail to the orch strand (decision record D6); the orch notice queue is no longer written by any caller, and card 9 removes it.

`NotifyPrime` keeps its role as the one way a module notifies the prime's orch.
It resolves the prime, then calls `notifyPrime` with `PrimeOrchStrand` for that prime, the caller's line, `$LYX_STRAND_NAME` and clients built over `creelcli.NewStrandClient` and `creelcli.NewProgramClient(prime, program)`.
`PrimeOrchStrand` reads the GUID the prime's orch state records and resolves its name through the prime's persisted reed state at each call; with no GUID recorded it answers not recorded, the same `Strand != ""` test batten's `orchStrandRecorded` and loom's `primeOrchRecorded` gates keep, and a GUID reed no longer holds answers recorded under the fixed `<shortname>:orch` name, so creel decides: it delivers to a listed strand of that name and refuses an unlisted one, and the refusal reaches the caller's warning with the line.
`notifyPrime`:

- with no orch strand recorded logs the notice at Info and posts nothing, answering `false, nil`;
- when the strand claim names a strand that is not an orch, posts with that strand claim, and when creel refuses it with `ErrStrandClaimOutsidePane` (matched with `errors.Is`, never by text) posts once more as the program;
- in every other case posts as the program with no strand claim;
- returns any other refusal or failure, with no retry, as an error carrying the notice's full line and creel's refusal, so the caller's warning holds the line for the operator to recover.

Bound: the program retry happens only on that one refusal, so a caller in a non-orch pane never signs as a program, and the caller walks no process ancestry of its own; creel's identity rules are unchanged.
A notice stays informational and changes no control flow: each caller warns on an error as today.
Notices keep their text, the `[batten notice]` prefix included, as the mail body.

Callers, each signing as its module's own program constant:
batten's `rowNotices` and the InnerRun `Notify` in `battenCLI.wire` answer `NotifyPrime`'s `posted` as their `queued`, and `loadedTacklerDeps`' shuttle notifier posts through it (`BATTEN`);
loom's runner notifier in `loomCLI.wire`, the driver hand-back in `goDriverDepsFor` and the landing `Notify` in `landingDeps` (`LOOM`);
burler's and webster's runner notifiers in `wireHub` (`BURLER`, `WEBSTER`), which carry shuttle's hold and dialog notices;
hibernate's `Notify` in `hubDeps` (`HIBERNATE`).
`PrimePaths`' comment drops the notice queue, and so do the comment on `landingshed.Deps`' `Notify` field ("whatever the notice queue says") and `handBackRun`'s comment and log line ("queued nowhere" becomes "posted nowhere").

The orch role stencil's wake-up becomes the creel reminder: the orch reads its mail with the command the reminder names, `lyx creel read --from <owner>` under a hold and `lyx creel read --all` otherwise, and handles each `[batten notice]` body as before; its landing fallback reads unread mail with `lyx creel read` and, for a notice already read or returned undelivered, its line in `lyx creel log --to <orch strand>` in place of `.lyx/notices.log` (D8).
No stencil gains a line letting an agent defer reading its mail.

Tests:
`TestNotifyPrime_SignsAndPosts`, a new untagged test, drives `notifyPrime` over fake clients and a fake orch lookup, with rows: no orch strand recorded posts nothing and answers not posted; a non-orch strand claim creel accepts posts once with that claim; a strand claim refused with `ErrStrandClaimOutsidePane` posts once more as the program; an orch strand claim and an empty claim post as the program only; another refusal posts nothing more and returns an error holding the line and the refusal.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/orchcli` by one.
`TestNotifyPrime_QueuesInThePrimeOnly`, in the integration tier because it needs a real hub, keeps its rows over that hub and asserts `PrimeOrchStrand` from a task worktree: the prime orch strand's name for a GUID the prime's reed state holds, not recorded with no GUID, and `<shortname>:orch` for a GUID reed does not hold; its file header names `PrimeOrchStrand` in place of the notice queue, and `assertNotices` goes.
`TestWire_NotifyWritesNothingWithoutAnOrchStrand` asserts `Notify` answers not queued with no error and writes nothing under the prime's `.lyx/orch`; `TestWire_NotifyQueuesOnThePrimesOrch` goes, since `notifyPrime`'s rows cover the post.
`TestLandingDeps_ConfigNoticeSeamsOverRealHub`'s Notify subtests assert a nil error and no notice file with no strand recorded, and with a strand recorded an error holding the notice line, since no creel daemon answers under go test.

Docs: `internal/orchcli/doc.go` ("Reaching the orch from another module", and the run loop's wake-up step), `internal/hibernatecli/doc.go` and `hibernatecli`'s `Command` help name creel mail in place of the notice queue; `docs/overview.md`'s creel row drops "No other module sends through creel until `creel-notices` lands" and states that notices and the orch's typed lines go through creel, and its shuttle and hibernate rows name creel mail in place of the orch notice queue.

**ImpactSummary:** Re-signs NotifyPrime and moves all nine notice call sites in batten, loom, burler, webster and hibernate onto it; each caller's control flow is unchanged, and two wiring tests drop their notice-queue assertions.
