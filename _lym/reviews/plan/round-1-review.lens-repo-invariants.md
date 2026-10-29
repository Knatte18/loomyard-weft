```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card, I checked test seams and tier tags, doc text the card falsifies, and whether the card can pass its own gate.

**Per-card list**

- **Card 1 (hook-output-seam)**
  - Seams: held. No `//lym:seam` or reflective field test covers `hookCommand`, `HookInput` or the renderer interface. `Claude` is the only implementer.
  - Docs: held. Every doc that names the block is listed (`shuttleengine/doc.go`, `shuttleengines/doc.go`, `claudeengine/doc.go`, `creelcli/doc.go`, `creelengine/doc.go`, `docs/overview.md`).
  - Order: held. `hookBlock` keeps its signature, so the hook-read subtests compile.
- **Card 2 (waiter-rewake-labels)**
  - Seams: held. The two `omitempty` fields have no reflective test.
  - Docs: held (`claudeengine/doc.go` is listed). Card 4 owns the orch stencil break below.
  - Order: held.
- **Card 3 (ack-failure-releases)**
  - Seams: held. `FailJournalWrites` goes in `export_test.go`, which is tagged `tier0`. `writeFault` is unexported and has no reflective test.
  - Docs: held (`creelengine/doc.go` is listed).
  - Order: held.
- **Card 4 (ring-claims-mail)**
  - Seams: held. Nothing reflects over `Request`, `Waiter`, `Outcome` or `Reading`.
  - Order: held. `HookWait` has one production caller (`server.go:307`) and one test caller (`deliver_test.go:1701`), both on the card. The unmarked hook-wait at `server_test.go:1075` is unaffected.
  - Docs: **broken**, finding 1.
- **Card 5 (client-ring-ack)**
  - Docs: held. `creelengine/doc.go` is listed. `pattern/PATTERN-daemon-wakeups.md:28` says `hook-wait` "reconnect[s] from one second", which stays true until a ring reaches the handler.
  - Order: held.
- **Card 6 (hook-wait-hands-mail)**
  - Seams: held. `TestHookWaitExit` goes in `hookread_test.go`, which is `tier0`.
  - Docs: **broken**, finding 1.
  - Order: **broken**, finding 2.
- **Card 7 (ring-echo)**
  - Seams: held. Nothing reflects over `Store` or `Response`.
  - Docs: held (`creelengine/doc.go` is listed).
  - Order: held. No `TestCreelCLI` subtest runs `hook-read` after an acked ring before card 8, so card 1's unspecified `HookOutput(event, "", "")` is never reached.
- **Card 8 (hook-read-renders-echo)**: seams held; docs held (`creelcli/doc.go` and `claudeengine/doc.go` are listed); order held.
- **Card 9 (claim-delivery-log)**
  - Seams: held. `TestStore_ClaimLog` is `tier0` and not parallel because it touches global logger state. Parallel top-level tests resume only after it finishes, and the parallel row `RepeatedSendErrorLogsOnceUntilItChanges` (`deliver_test.go:1323`) counts only its own strings.
  - Docs: held. Order: held.
- **Card 10 (smoke-hook-delivery)**: seams held (the test file is `tier3`); docs held; order held.

**Findings**

1. **MEDIUM, consistency.** `contracts/stencils/orch/orch-template-role.md:41`, falsified by cards 4 and 6, and no card lists it.
   - The stencil tells the orch agent: "An idle orch is woken first by the fixed line `[creel] Mail is waiting for you.`; end that turn, after which the mail shows."
   - After D3 and D4, the waiter wakes the idle orch with `[creel] Mail that arrived while you were idle: &lt;lines&gt;`, so the mail is already there. Telling the agent to end that turn works against D3's goal that the woken turn acts on its mail.
   - Line 40, "arrives … under a tool result or as your turn ends", also leaves out the wake.
   - The bell, typed only when no waiter reaches the orch, is now the exception.
   - The decision record's In-scope list names package docs and code comments but not this stencil. A fix may need the parent to license it under the CLAUDE.md rule that agent-facing rules go into stencils.
   - Evidence: `orch-template-role.md:40-41`; `deliver.go:30` (`BellLine`); `hookwait.go:67`.
2. **MEDIUM, consistency.** Card 6 (`internal/creelcli#TestCreelCLI`) cannot pass its batch gate (tier1) as planned.
   - The subtest "a hook-wait superseded by a newer one of the strand exits 0 printing nothing" asserts `newer.String() != creelengine.BellLine+"\n"` at `cli_test.go:1011`.
   - After card 6, the newer waiter writes the claimed lines of #1 and #2 instead.
   - Card 6's Intent changes only the "rings a strand as exit 2" subtest (line 935) and adds one subtest, so this assertion has no planned change.
   - Evidence: `cli_test.go:981-1014`.

**Tool calls**

1. Searched for `lym:seam` and reflective tests, and for docs naming the old behaviour: to find seam tests and falsified docs beyond the manifest.
2. Searched `PATTERN.md`, `pattern/`, other `doc.go` files and `contracts`: for hook-delivery text the manifest did not cover.
3. Read `PATTERN-daemon-wakeups.md:24-32` and the `orchengine/doc.go` lines: to judge whether the hits are falsified.
4. Searched shipped stencils and skills for wake and bell text: they were not in the manifest.
5. Read `orch-template-role.md:32-46`: for the context of the stencil hit.
6. Searched for log-capture assertions and `HookWait` callers, and read `deliver_test.go:1316-1345`: to check card 9's global-logger interaction and card 4's caller set.
