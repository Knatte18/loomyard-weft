```yaml
lens: existing-paths
outcome: reported
```

Existing-paths review of decision-record.md: I traced the code that already runs and writes, resets or reads each state the decisions add or change.

**Paths traced, per decision**

- **D1**:
  - `orchcli.NotifyParent`/`NotifyFrom` walk; `OrchLive` (reed config plus status) runs from an agent pane's cwd. Survives.
  - The notifier wiring in burlercli, loomcli, webstercli and battencli `primeShuttle` resolves the same walk. Survives.
  - The LLM driver starts from the pair (`loomcli/start.go`), so its `--parent` and the loom notifier agree. Survives.
- **D2/D6**:
  - The parent directive is rendered by every `parentdirective.Directive` caller; tackler and arbiter render the no-parent variant. Survives.
  - The appended sentence is written at launch and resume. Survives; a session launched before the change keeps the old sentence until its next resume.
- **D3**: the hold notice goes through `runner.SetNotifier` in all four wirings. Survives.
- **D7**:
  - `seatengine/prompt.go` adds the requirement and `seatengine/resume.go` rebuilds the spec.
  - `Spec.Requires` is not persisted in the run record, so an in-flight table that is attached again never meets the removed name. Survives.
- **D8**:
  - Pause handling: `shuttlecli/send.go` reads the pause record and refuses. `lym creel inject` queues under a pause and types after the resume, with the TTL frozen. Breaks the record's description; see finding 3.
  - `shuttleengine.WithMessageTail` is applied by send.go and the hold notice names `MessageTail`, so the tail is now typed by the orch by hand. Survives.
- **D9**: the mark is a new journal key on `posted`. The tolerant fold ignores unknown keys, and a replaced binary ends the daemon. Survives.
- **D10, removed-holder sweep**: the sweep runs and acts only on unread mail or queued injects. Breaks; see finding 1.
- **D10, worker retire**: `hasPending` counts only unread mail and queued injects. Covered by the new per-receiver timer the record adds. Survives.
- **D10, pause, hold and liveness clock across a restart**: inject TTL survives a restart only through `aged` records, and `ageInjects` ignores holds. Breaks; see finding 2.
- **D10/D12, retention `retain`**: it carries only unread messages, queued injects and live holds. Breaks; see finding 4.
- **D11**: the `log --id` listing of `reminded` and `carried` events gains `chased` and `unanswered`. Survives.
- **D12**: `GoClient.Log` exists; the kept window is `journalRetention`. Survives, except for the retention edge in finding 4.

**Findings**

1. **BLOCKING · design · D10 ("A holder that the removed-holder sweep finds gone makes the message `unanswered` at once")**
   - Summary: the sweep never runs for, and never acts on, a read message, so a holder that dies after reading is never found gone. The chase clock stays frozen ("counts only while the holder is live"), and the message is neither chased nor `unanswered`.
   - Evidence:
     - `internal/creelengine/sweep.go:36` "sweepLoop sweeps while unread mail or a queued inject exists".
     - `sweep.go:173-179` `sweepNeeded` and `sweepSubjects` cover unread mail or queued injects only.
     - `sweep.go:202` only unread messages go back along their path.
   - The record does not say that the sweep's trigger and subject set widen to read, marked, unanswered messages.

2. **MEDIUM · design · D10 ("The daemon derives the chase state from the journal (read time, `chased` records)" and "the rule creel already applies to an inject's TTL")**
   - Summary: the inject TTL is not derived from post time. Its consumed time is journaled as `aged` records every minute, and it stops only while the receiver is dormant or paused, never under a hold.
   - Consequence: a frozen chase clock can't be rebuilt from the read time and `chased` records after a restart, so a restart after a pause or hold chases early. The "same rule as inject TTL" claim is also false for the hold condition.
   - Evidence:
     - `internal/creelengine/deliver.go:22` `agedPersistInterval = time.Minute`.
     - `deliver.go:555-575` `ageInjects` adds elapsed time with no hold check.
     - `deliver.go:603-617` `persistAged`.
     - `journal.go:91-92` `ConsumedSeconds`.

3. **MEDIUM · design · D8 Rationale and Bound ("keeping them as wrappers would leave two operator routes into a pane, one of which bypasses creel's journal, hold and pause rules"; Bound names only the lost guid addressing)**
   - Summary: `lym shuttle send` does apply the pause rule. It refuses under a pause and names the resume. The replacement `lym creel inject` queues instead and types the line after the resume.
   - Consequence: an operator's line written during a pause now lands later, possibly stale. The Bound doesn't state this, and the record misreads the existing path.
   - Evidence:
     - `internal/shuttlecli/send.go:44-52` reads `pauserecord.ReadCoverage` and refuses.
     - `pattern/PATTERN-pane-text-injection.md`, "A pause holds prompts": "`lym loom resume` and `lym shuttle send` refuse with the pause's way forward"; "Creel holds the inject".

4. **MEDIUM · design · D10 and D12 (journal-derived chase state; "over the creel journal's kept window")**
   - Summary: retention carries only unread messages, queued injects and live holds. A read, marked message whose clock is frozen past seven days (a dormant or paused holder) has its `posted`/`read` lines deleted with no carry. Its chase state is lost: no chase, no `unanswered` record, and a waiting `--wait-reply` is never answered.
   - The same deletion leaves D12 counting replies and chases whose `posted` record has already aged out. The record states neither edge.
   - Evidence:
     - `internal/creelengine/journal.go:58` `journalRetention = 7 * 24 * time.Hour`.
     - `journal.go:285` "retain appends a carried record for every unread message, queued inject and live hold".
     - `journal.go:315` `w.pool.prune(cutoff)`.

Tool calls:
1. `sweep.go` trigger lines and `Store.hasPending`: to check whether the sweep and the workers run for read messages. The context held only the creel doc's prose about the sweep.
2. `Requires` usage under `internal/shuttleengine`: to check whether a persisted run record carries the removed requirement through attach or resume. The context had only `requires_test.go`'s attach refusal.
