```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. For each function or branch the diff adds or changes, run one concrete case and say whether it is correct.

**Per function or branch, the case I ran, correct or not**

- `blockingEscalationDetail` (bouncer.go:1558–1578)
  - BlockingEscalates off → "". Correct.
  - Ledger unreadable → "ledger could not be read" detail. Correct.
  - Keys `[hard]`, verifier closes `hard` → "" → AutoAccept → `acceptRound`, and the carry-over reads the rewritten ledger from disk. Correct.
  - Keys `[a,b]`, verifier closes `a` → detail names `b`; a re-call re-reads the ledger and gets `[b]`. Correct.
  - Verifier fails → all keys plus the note. Correct.
- `escalateUndecided` with leftOut=decision: the verifier closes `hard` → "" → `escalateDecision`, whose `decisionRefusal` gets the stale in-memory ledger. For CONTINUE/CIRCLING that only runs `droppedKeyRefusal`, and a status change drops no key. Correct.
- `closeVerifiedBlocking`
  - Escalation record present → keys, "". Correct.
  - Malformed escalation record → skip, then `escalateWith` handles the record. Correct.
  - No fixer report, `Spawned` set, disputed disposition → skip. Correct.
  - `fixLeftTargetUnchanged` true → skip. Correct.
  - Verifier closes none → keys, "". Correct.
  - Verifier closes a key it was not asked about → filtered out by `keys`. Correct.
  - `markLedgerResolved` fails → keys plus the note. Correct.
- `fixedBlockingFindings`
  - Fixed F1 BLOCKING → `[F1]`. Correct.
  - Fixed MEDIUM finding → none. Correct.
  - Unparseable review or report → none. Correct.
- `fixLeftTargetUnchanged`
  - Equal ids → true.
  - Missing record, empty id or nil Snapshot → false, so it verifies. Correct.
- `verification`
  - Parsing file on disk → reused.
  - Unparseable file on disk → `runVerifier` archives it and respawns.
  - Read error other than not-exist → error.
  - Verifier writes nothing → "wrote no verification" error. Correct.
- `runVerifier`
  - Attach error → error.
  - Not attached → archive, then Run.
  - Run error → error.
  - Run ends non-done with no file → caught by the read that follows. Correct.
  - Usage is recorded as a judge pass: see finding 3.
- `RenderVerifyPrompt`: keys `[stand-in-key]` and F1 fill every marker. Each read failure is wrapped. Correct.
- `parseVerification`
  - No frontmatter, bad YAML, other round, empty key, status `maybe` → error.
  - Empty entries → empty map.
  - The same key with `closed` then `open` → closed: see finding 4.
- `renderLedger`: entries, rounds and round are kept. Empty class, severity and fix are left out, which `parseLedger` accepts. Prose is kept. Correct.
- `markLedgerResolved`
  - Open `hard` in `closed` → resolved. Other entries and already resolved entries are unchanged. Correct.
  - Write is not atomic: see finding 1.
- `rejudge`: archives the three judge outputs plus the verification. `append` on a fresh slice of cap 3 is safe. Correct.
- `ledgerEntryYAML` omitempty: affects only marshaling. Parsing is unchanged. Correct.

**Findings**

1. **LOW, design**: `markLedgerResolved` overwrites the judge's committed ledger in place with `os.WriteFile`, which truncates first.
   - Location: internal/shedadapters/bouncerverify.go:829.
   - A crash or a failed write mid-way leaves a truncated ledger, and nothing archived the original.
   - The next Call then hits "the round's ledger could not be read" (bouncer.go:252), and the judge's only record is gone.
   - Evidence: bouncerverify.go:825–831 has no archive and no temp-file rename before the write.
2. **LOW, design**: the hard-cap halt path archives `judgeOutputs` but not `round-<N>-verification.md`.
   - Location: internal/shedadapters/bouncerdecision.go:210.
   - The resume re-judges the round, and `verification()` then reuses a ruling made on the retired ledger's keys without a fresh pass.
   - That is the exact case the diff gives as its reason for archiving the verification in `rejudge`.
   - Evidence: bouncerdecision.go:210 against bouncer.go:1220 and bouncerverify.go:713–716.
3. **LOW, design**: `runVerifier` records the verifier pass with `recordPass(round, false, result)`.
   - Location: internal/shedadapters/bouncerverify.go:766.
   - That call appends it to the round usage record's judge list, so the record shows a second judge pass for a verifier run.
   - Evidence: bouncer.go:1013–1024, where only the seed/judge split exists.
4. **NIT, design**: `parseVerification` accepts a key that appears twice.
   - Location: internal/shedadapters/bouncerverify.go:616–619.
   - A `closed` entry wins over a later `open` entry for the same key, so a self-contradicting ruling closes the ledger entry instead of being refused as malformed.
   - Evidence: the loop sets `closed[entry.Key] = true` and never clears it.

**Tool calls**

1. `decisionRefusal` and `renderDecisionDetail` in bouncerdecision.go: to run the decision-escalation path with the stale in-memory ledger after the verifier closes keys. The manifest had not read these.
2. `refusalLeavingOut`, the judge stencil's ledger format, and the `judgeOutputs` callers: to check what the ledger rewrite drops and which archive sites leave the verification behind. The manifest read only heads and ledger lines.
3. The judge-spawn archive sites at bouncer.go:2062/2157 and readers of the usage judge list: to check whether a verifier pass recorded as a judge pass, or a missed archive, changes behavior. Not in the manifest.
