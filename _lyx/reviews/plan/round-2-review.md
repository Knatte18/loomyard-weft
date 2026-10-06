---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/05-rerun-removes-superseded-strand.md
    summary: Card 5 extends the strand teardown to probe-only callers that start no run, which D4 does not license
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/13-orch-reload-sequence.md
    summary: Card 13 leaves open where the plugins step advances, and the existing advanceReload helper would type the next step on the same tick
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/11-watcher-reloads-fresh-compaction-only.md
    summary: A boundary held in w.compactedAt survives a strand change because initCursor does not clear it and the card does not add that
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/10-compaction-boundary-turn-ends.md
    summary: ReadTurnEndAfter is true when the Stop payload carries no message, so an older read turn end plus one unread newer one still reloads, against D8
---

### [MEDIUM] Strand teardown reaches callers that start no run

**Location:** `_lyx/plan/05-rerun-removes-superseded-strand.md`, the paragraph beginning "Every caller whose probe answers not found inherits the teardown".
**Issue:** D4 removes a predecessor's strand "when a producer row starts a fresh shuttle run after its attach probe found nothing attachable".
The card puts the teardown on the final not-found return of `Runner.AttachGated`, so the Bouncer's `awaitLiveJudge` and `awaitLiveSeed` and the reflection probe in `internal/frictionengine/reflect.go` also tear down strands although none of them starts a run afterwards.
The card says so itself and argues it is harmless.
That is scope the decision record does not license, and it adds a destructive routing edge on read-only probe paths.
D4's bound ("a start the probe refuses removes nothing") is stated for starts only.
**Fix:** Restrict the teardown to the callers that go on to start a fresh run: the removal runs from the start path (an explicit step or option the starting callers use), and the probe-only callers keep a pure probe.
Rewrite the paragraph, the ImpactSummary, the `AttachGated` doc-comment instruction and the test rows so a probe-only call removes nothing, and add a row asserting that.

### [LOW] Plugins step advance is underspecified and the helper at hand does the wrong thing

**Location:** `_lyx/plan/13-orch-reload-sequence.md`, the plugins-step bullet and the `typeReloadStep` sentence.
**Issue:** The card says the plugins step persists the move to the next step and returns without typing anything else on that tick, and separately that `typeReloadStep` gains a plugins case.
It does not say that the move lives in that case, which both `startReload` and a restarted `tickResuming` reach, nor that the shared tail `return w.confirm(st)` must not run for it.
The helper that already persists a move, `advanceReload`, ends by calling `tickResuming`, which probes idle and types the next step on the same tick, the behavior D10's bound forbids.
An implementer following the card literally can pick either.
**Fix:** State that the plugins case in `typeReloadStep` persists the move itself (next step by `ReloadSkipsSkills`, `ReloadTypedAt` zero, offset at the cursor) and returns without `confirm`, and that it does not call `advanceReload`.
The restart test row then also asserts nothing but `reload-plugins` is typed on the retyping tick.

### [LOW] Held boundary survives a strand change

**Location:** `_lyx/plan/11-watcher-reloads-fresh-compaction-only.md`, the held-boundary paragraph.
**Issue:** The card clears `w.compactedAt` only on a tick that read a turn end.
`initCursor`, run when `w.strand != st.Strand`, resets `newest`, `seen` and `replaying` but not `compactedAt`, so a boundary an earlier tick held for the old strand stays held; a later tick that reads no turn end then lets `tickIdle` type a reload into the new strand.
The first focus question asks whether any path lets a held boundary reach `tickIdle` unevaluated, and this is the one the card leaves open.
The card's stated rule for ticks that read a turn end is otherwise sufficient, and its fourth test row covers it.
**Fix:** Add `w.compactedAt = time.Time{}` to the state `initCursor` resets, state it in the card, and extend the held-boundary test with a strand change between the holding tick and the next.

### [LOW] Missing Stop message passes the read-turn-end check

**Location:** `_lyx/plan/10-compaction-boundary-turn-ends.md`, the paragraph defining `ReadTurnEndAfter`.
**Issue:** `ReadTurnEndAfter` is true when the newest turn end's text equals the payload's `last_assistant_message`, "or the payload carries none".
With no message there is nothing to compare, so a watcher that read an older turn end, with one unread newer turn end after the boundary, gets count 1 and read true and types the reload.
D8 requires the single later turn end to be the one being read.
**Fix:** Decide the no-message case explicitly: treat it as not confirmed (false) so it types nothing and leaves the baseline until a readable turn end arrives, and add a row to the `usage_test.go` list.
If the plan keeps the lenient reading, state in the card why a missing message cannot occur at a turn end the watcher reads.

## Focus checks

- Watcher reload gating: a tick that read a turn end re-sets `w.compactedAt` from its own evaluation (confirmed boundary or zero), the existing tail `if !compactedAt.IsZero() { w.compactedAt = ... }` is replaced by that rule, and the fourth test row drives a held boundary into a tick that reads a second turn end.
  Verdict: no path survives except the strand change in F3 and the no-message case in F4.
- Orch reload sequence: Card 13 handles the plugins step in `tickResuming` before the skills branch and gives it a `typeReloadStep` case, so a restart at the plugins step resumes there as its own step, including with a `ReloadTypedAt` older than the handoff timeout and a turn end read; the restart test row covers it.
  Verdict: not a defect, apart from the advance ambiguity in F2.
