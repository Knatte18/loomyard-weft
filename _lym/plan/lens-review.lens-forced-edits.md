```yaml
lens: forced-edits
outcome: reported
```

Task: forced-edits review of the plan in `_lym/plan`. I checked callers, tests, docs and registrations that the cards' changes force.

**Findings**

1. **LOW — Card 11, D2 (no-strand calls now wait for slots without a cap).** `internal/gateslot/doc.go:51` says "the slot wait is bounded by the keep-cache-warm cap with no knob". Card 11 makes that false. No card rewrites the line: cards 9 and 14 only list the file under Extends. Card 11 should list `//internal/gateslot/doc.go` under Edit and reword that sentence.

2. **LOW — Card 11, SlotBusyExit's changed meaning.** Card 11 never mentions three texts that still give 75 its old meaning:
   - The `internal/gatecli#SlotBusyExit` doc comment (`internal/gatecli/cli.go:21`) still says "returned when no slot frees within the keep-cache-warm cap". After card 11 an in-process call never returns 75; only a capped front does, meaning "the run goes on".
   - The doc's `# Exit codes` paragraph (`internal/gatecli/doc.go`, "SlotBusyExit (75) for a busy hub when no earlier run failed") describes the same old meaning. Card 11 edits doc.go, but its Intent names only the Slots section and the new Capped call section.
   - `runTest`'s doc comment (`run.go`, "SlotBusyExit for a hub whose slots stay held past request.slotWait") names a field card 11 removes.

   Card 11 should list `internal/gatecli#SlotBusyExit` and `internal/gatecli#runTest`'s comment (runTest is already listed for card 9's work) and reword the Exit codes paragraph.

3. **LOW — Card 11, dead branch.** `internal/gatecli#haltExitCode` has the case `end.code == SlotBusyExit && firstFailureCode != 0`. It only fires when `acquireSlot` answers a busy hub. Card 11 removes that answer (`answerSlotBusy` is deleted and the in-process wait is uncapped), so the branch can never run, and its doc comment ("A busy hub after a failed run exits with that failure's code") goes stale. No card lists `internal/gatecli#haltExitCode`. Either drop the branch on card 11 or state why it stays.

4. **LOW — Card 11, operator docs.** `docs/dotnet-hub.md:126` says a direct call "passes `--wait` so its drain is not dropped at the keep-cache-warm cap". After card 11 a no-strand call has no cap, so the stated reason is wrong. Card 11 lists `docs/overview.md` but not `docs/dotnet-hub.md`.

Tool calls:
- Searched for `slotWait`, `RunLine`/`RunCards` callers, `SlotBusyExit`/`slot_free` in tests, `fakes_test.go`'s package clause, `--wait --gate` test strings, the gate-rule texts, the CLI-tree hidden-verb rule and the `cardverify.Runner` constructions. My context lacked these sets of callers and tests.
- Read `internal/websterengine/cardverify.go` and `recordbatch.go:525-545`, and searched for tests and markdown naming the gate rule or `slot_free`. Needed to confirm `rerunCardVerifies` is `RunCards`' only production caller and to find who pins the gate-rule text.
- Read the gate-rule assertion in `claudeengine/settings_test.go` and the gate paragraph of `docs/overview.md`. Needed to check whether card 15's text change breaks a test outside shellguard; the assertion only pins the rule's prefix, so it does not.
- Read the claude rows of `configsync_test.go` and `hubreconcile/ensure_integration_test.go`, and checked for an import allowlist in `shuttlecli`. Needed to check whether card 2's new template keys break those template-text tests; their claude cases only rewrite the `prompt_cache_ttl_roles` map, so they do not.
