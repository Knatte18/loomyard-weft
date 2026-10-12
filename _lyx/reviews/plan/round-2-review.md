---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/14-landing-watchdog-halt.md
    summary: Card 14's watchdog clause reaches Finalize's failed-verify halt reason, but the card gives the case a refusal-spec row only for Merge-In and Publish, and no row covers Finalize's failed verify
    fix: >-
      In card 14's Intent, extend the contracts/specs/refusal-spec.md paragraph so Finalize's failed verify also gets a row carrying the watchdog case. Either add a row "Finalize: verify failed" in the landing rows beside "Merge-In: verify failed" and "Publish: verify failed", naming the reason verdict gives Finalize (fix forward on the task branch, then resume, or the told way forward when the failure record was written) plus the watchdog case (the window, and raising busy_quit_sec through "lyx config gate" from the prime for a package that is CPU-bound but healthy), or widen the "Merge-In: verify failed" row's label and trigger to Merge-In or Finalize if their way forward is the same. Name the chosen row in the card text. No target changes; refusal-spec.md is already on the card.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/07-pattern-tolerant-records.md
    summary: loomshed's rework round record.json is a Go-written JSON run record read and rewritten through raw encoding/json, so a newer binary's unknown key in it is dropped, and card 7's PATTERN entry lands with this violator in place
    fix: >-
      Add `internal/loomshed#writeRoundRecord` under card 7's **Edit:** (checked with lyx loom validate-plan --overlay, no new blocking finding), and add an Intent paragraph saying writeRoundRecord writes through state.WriteJSON with a lock path beside the record (for example recordPath + ".lock"), which keeps unknown keys by D2 and still replaces the file atomically. finishRound's read-then-rewrite then keeps a newer binary's keys. The existing prrework tests keep pinning the record's content, so no new test is needed. Alternatively, place the same edit in a new card inserted with lyx loom insert-card after card 3 and before card 7.
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/plan/11-gate-test-watchdog.md
    summary: Card 11 sends the watchdog line to the verb's stderr, but runTest receives only cmd.OutOrStdout() and the card names no route by which runTest gets a stderr writer
    fix: >-
      In card 11's Wiring list, state the route. testRequest (already an Edit target) gains a stderr io.Writer field, newCommand's RunE fills it from cmd.ErrOrStderr(), and runTest hands request.stderr to the watchdog as its writer. Do not add a runTest parameter, which would need a re-sign arrow on internal/gatecli#runTest. No target changes.
---

### [BLOCKING] Finalize's failed verify gains the watchdog case with no refusal-spec row

**Location:** `_lyx/plan/14-landing-watchdog-halt.md`

**Issue:** Card 14 changes `internal/landingshed#verifyGate.verdict`, which serves the plan verify of all three landing producers.
`internal/landingshed/finalize.go:409` calls `fz.gate.check(...)`, which ends in `verdict` exactly as `mergein.go:182` does, and the card's own ImpactSummary says the verdict is "shared by the plan verify and `publish_verify` of all three landing producers".
So after this card, Finalize's halt reason also names the watchdog window and `busy_quit_sec` raised through `lyx config gate`.
The card's refusal-spec edit covers only "Merge-In: verify failed" and "Publish: verify failed".
`contracts/specs/refusal-spec.md` has no row for a failed verify in Finalize: its Finalize rows cover a dirty tree, a merge in progress, an unstopped conflict session and a failed parent-records commit, never a failed verify.
D6 requires "the landing rows for a failed verify" to gain the way forward in the same commit.
`PATTERN-refusal-way-forward` requires every reachable refusal to have its row and a new refusal to land with that row.
The card therefore lands a new halt case, Finalize after a watchdog quit, with no row naming it.

### [MEDIUM] The rework round record is rewritten outside `internal/state`

**Location:** `_lyx/plan/07-pattern-tolerant-records.md`

**Issue:** The decision record's goal is that "every rewrite keeps the fields its binary does not know".
D9's entry, which card 7 writes, says a rewrite of a Go-written JSON record goes through `internal/state` or through a type that keeps its own keys.
`internal/loomshed/prrework.go` violates both.
`readRoundRecord` decodes `record.json` with `json.Unmarshal`.
`finishRound` then rewrites the same record carrying the class through `writeRoundRecord`, which uses `json.MarshalIndent` and a temp-file rename.
`roundRecord` keeps no unknown keys, so a key that a newer binary added to `record.json` is lost on that rewrite.
That is the incident's failure class: a long-lived loom run drops a newer binary's field.
No card touches `prrework.go`, so card 7's PATTERN entry lands already false for a production file.
The plan's method found strict decoders by searching for `DisallowUnknownFields`, which matches only that call, never a raw read-modify-write.
The other raw `json.Marshal` plus `os.WriteFile` sites I read, such as friction's covered-notes record, `statuspane`'s sidecar and `steprecord`'s inflight record, write a fresh value and never decode then rewrite a record, so `writeRoundRecord` is the one site that needs the fix.

### [LOW] The watchdog's stderr writer has no route into `runTest`

**Location:** `_lyx/plan/11-gate-test-watchdog.md`

**Issue:** D7 says the verb prints the watchdog line "to its stderr".
Card 11's `watchdog` holds "the writer the line goes to (the verb's stderr)", and the integration row asserts the line on the gate process's stderr.
`internal/gatecli/cli.go`'s RunE calls `runTest(cmd.Context(), cmd.OutOrStdout(), request)`.
`runTest` has no stderr writer, and go test's stdout and stderr both go to `out` (`internal/gatecli/run.go`, `cmd.Stdout = out`, `cmd.Stderr = out`).
The card lists no `runTest` re-sign and no `testRequest` field for it, since `testRequest` gains only `samplePeriod`.
An implementer must therefore choose between `os.Stderr`, which an in-process `RunCLI` capture never sees, a new `runTest` parameter, which needs a re-sign arrow the card lacks, and a request field.
The plan leaves that choice open.

## Rejected

- Card 3's merge-then-indent layout: `json.MarshalIndent` is `json.Marshal` followed by indentation, so marshalling compactly, merging and then indenting with two spaces gives the same bytes as today for a value with no kept keys, and card 1 pins `board.json`'s Marshaler output.
- Budget rows: the current counts match each card's arithmetic: `internal/state` 8, `internal/proc` 17, `internal/gateslot` 11, `internal/gatecli` 2 against a row of 3, `cmd/lyx` 78, `internal/shedrun` 23, `internal/websterengine` 202.
- Card 7's headroom arithmetic: the four shortened lines lose 285 characters and the new entry adds 177 plus its newline.
- `PATTERN-daemon-wakeups` for card 11: the sampler spawns nothing, so the backoff rule does not bind it, and its 5-second and injected 1-second periods respect the one-second floor.
- `internal/parentreview/store.go`, named in the decision record's notes: it decodes with plain `json.Unmarshal` and writes through `state`, so card 3 covers it.
- The loom verify gate also serves Merge-Burler-Fix and Darn-Burler-Fix, which card 13's Intent does not name. The Burler-Fix rows' exhaustion routes to the fix row's recovery rather than a halt, and the renderer change is harmless there, so this is not a finding.
