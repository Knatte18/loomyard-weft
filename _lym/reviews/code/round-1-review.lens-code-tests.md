```yaml
lens: code-tests
outcome: reported
```

Task: as the code-tests lens, map each behaviour the diff adds or changes to the test row that would fail if it broke, and report the ones nothing covers.

**Behaviours and the tests that pin them**

1. A confirmed fix closes its entry and the round accepts: `TestBouncer_FixVerification/AConfirmedFixClosesTheEntryAndTheRoundAccepts`.
2. A verifier ruling of `open` escalates with cause budget, naming the key: `/AFixTheVerifierKeepsOpenEscalatesNamingIt`.
3. A verifier that writes no file escalates with the failure note: `/AVerifierThatWritesNothingEscalatesNamingTheFailedCheck`.
4. `parseVerification` refuses an unknown status: `/AVerificationOfAnUnknownStatusEscalatesNamingTheFailedCheck`.
5. `parseVerification` refuses a verification naming another round: none.
6. `parseVerification` refuses an entry with an empty key: none.
7. A verification already on disk is reused with no new spawn: `/AVerificationOnDiskIsUsedWithoutAVerifier`.
8. A verification on disk that does not parse is replaced by a fresh verifier pass: none.
9. A disputed finding gets no verification: `/ADisputedFindingIsNotVerified`.
10. A round with no fixer report gets no verification: `/ARoundWithNoFixerReportIsNotVerified`.
11. A fixer report the engine wrote for a fixer it never spawned (`Spawned != nil`) gets no verification: none.
12. A `fixed-differently` disposition triggers verification: none. Every row uses `fixed`.
13. Only BLOCKING findings the fixer reported fixed reach the verifier; lower severities are left out: none. The fixtures fix only the BLOCKING finding F1.
14. A fix that left the target unchanged gets no verification: `/AFixThatLeftTheTargetUnchangedIsNotVerified`.
15. A round that already has an escalation record spawns no verifier on a re-call: the re-call assertion `verifier runs after the re-call` in every Awaiting row.
16. A mixed result, where some keys close and the rest stay open, so the parent hears only the open ones: none. Every row's ledger has exactly one open BLOCKING key.
17. A failed spawn (`Shuttle.Run` or `Attach` errors) leaves every key open and puts the failure in the detail: none.
18. Attaching to a live verifier instead of spawning one: none.
19. A failed ledger write in `markLedgerResolved` leaves every key open and names the failure: none.
20. `markLedgerResolved` and `renderLedger` keep every other entry, the rounds, class, severity, fix and prose: none. The test checks only the `hard` entry's status.
21. `rejudge` archives `round-<N>-verification.md` along with the judge's outputs: none. No test outside `bouncer_verify_test.go` references `verificationPath`.
22. The verifier's pass is recorded in the round's usage record (`recordPass`): none.
23. The verifier spec's output file, model, config role, round and prompt content: `requireVerifierSpec`, called from every row that spawns a verifier.
24. A confirmed fix beside a decision finding goes to the decision escalation: `/AConfirmedFixLeavesADecisionFindingToTheDecisionEscalation`.
25. The verify stencil's markers: the `bouncer-template-verify` row in `TestStencils_TopLevelMarkers`.
26. The verifier launch is the same on both engines: the `Bouncer.runVerifier` row in `TestProducers_CodexManifestHoldsClaudes` (`internal/shuttleparity/producers_test.go:112`).
27. The `omitempty` tags on `ledgerEntryYAML` only matter through `renderLedger`, so they fall under item 20.

**Findings**

- **MEDIUM, design.** `internal/shedadapters/bouncer.go:1569` (`blockingEscalationDetail` → `closeVerifiedBlocking`). Nothing tests a partial close, so the contract that "only the keys still open reach the parent" is unpinned when there is more than one key.
  - Evidence: every row in `internal/shedadapters/bouncer_verify_test.go:350` builds a ledger with the single open BLOCKING key `hard`.
  - Fix: add a row whose ledger holds two open BLOCKING keys and whose verification closes one. Assert that the closed key reads `resolved`, the other stays `open`, and the detail names only the open key.

- **MEDIUM, design.** `internal/shedadapters/bouncer.go:214` (`rejudge`). The new archiving of `round-<N>-verification.md` has no test.
  - Evidence: a grep for `verificationPath`, `renderLedger`, `markLedgerResolved` and `parseVerification` across `internal/shedadapters/*_test.go`, excluding `bouncer_verify_test.go`, returned nothing.
  - Fix: in an existing rejudge test, write a verification file before the refused judgment and assert it was archived.

- **MEDIUM, design.** `internal/shedadapters/bouncerverify.go:813` (`markLedgerResolved` and `renderLedger`). Nothing asserts that the rewrite keeps the rest of the ledger: other entries, rounds, class, severity, fix and prose. This is a rewrite of a file that the soft-cap rule, the carry-over and the next judge all read.
  - Evidence: `bouncer_verify_test.go:409-419` checks only `hard`'s status. The fixture ledger also has an `alpha` entry, but nothing checks that it survives.
  - Fix: in the confirmed-fix row, compare the whole rewritten ledger against a literal, prose included.

- **MEDIUM, design.** `internal/shedadapters/bouncerverify.go:637`, the failure paths of `closeVerifiedBlocking`. Neither failure-path note has a test row:
  - a failed verifier spawn (`Shuttle.Run` or `Attach` errors): "the check … failed: …";
  - a failed ledger write: "the verified findings could not be closed in the ledger".

  Evidence: the only failure row is "writes nothing" (`bouncer_verify_test.go:332`). The fake's `RunFn` never returns an error.
  Fix: add a row where `RunFn` returns an error for the verifier spec and assert the escalation detail carries the failure note.

- **LOW, design.** `internal/shedadapters/bouncerverify.go:672` (`fixedBlockingFindings`). Three selection rules have no row:
  - `fixed-differently` qualifies for verification;
  - a lower-severity finding the fixer reported fixed is left out;
  - a report with `Spawned` set yields nothing.

  Evidence: the fixtures at `bouncer_verify_test.go:291-292` use only `fixed` and `disputed`, and F1 is the only finding they report fixed.
  Fix: add rows for `fixed-differently` (verifier runs) and `spawned: false` (no verifier). Add a MEDIUM finding marked fixed, and assert its id is not in the verifier's prompt.

- **LOW, design.** `internal/shedadapters/bouncerverify.go:599-625` (`parseVerification`) and `:711-719` (`verification`). Two behaviours have no row:
  - refusing a verification that names another round, and an entry with an empty key;
  - re-running the verifier when the file on disk does not parse.

  Evidence: only the unknown-status refusal is exercised (`bouncer_verify_test.go:295, 334`). Of the on-disk cases, only a valid file is (`:336`).
  Fix: add an `onDisk` row with a verification that names another round. Assert one verifier run and the outcome of the fresh file.

- **NIT, design.** `internal/shedadapters/bouncerverify.go:766` (`recordPass` in `runVerifier`). Nothing asserts that the verifier pass lands in the round's usage record.
  - Evidence: a grep for `usage` in `bouncer_verify_test.go` returned nothing.
  - Fix: assert the round's usage record holds a judge entry after a verifier run, or accept the gap as a log-level side effect.

**Tool calls**

1. A grep of `internal/shedadapters/*_test.go` for `verificationPath`, `verification.md`, `renderLedger`, `markLedgerResolved` and `parseVerification`, plus a grep of `bouncer_verify_test.go` for usage assertions. My context held only the new test file, so I could not tell whether other tests covered the rejudge archive, the ledger render or the usage record.
