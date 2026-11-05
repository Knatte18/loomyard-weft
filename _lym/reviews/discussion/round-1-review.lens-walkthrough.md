```yaml
lens: walkthrough
outcome: reported
```

**Decisions walked:**
- D1 — new pawl rows; Beam-Await `on_done` → Finalize. Purpose met.
- D2 — ready → dobby request → accept writes Finalize → `RunHolds` holds. Purpose met (hand `lym batten run` not walked; bound-breaker owns it).
- D3 — beam with a parent moved after merge-in, `ready`. Refusal 3 fires. Purpose met.
- D4 — nested beam forked from beam reads the landing record naming the other slug → no record. Purpose met.
- D5 — members land, describe. Today's writer already lists PR number, merge time and summary per commit (`internal/landingshed/beamdescribe.go:71`, `:185`). Purpose met.
- D6 — open, member lands, update, status `clean`. Purpose met.
- D7 — pr open, member lands, notice, `lym beam describe` regenerates summary.md. Purpose partly met (F4).
- D8 — recorded PR, parent moved after ready, Finalize merges parent in and pushes, GitHub merge. **Broken** (F2).
- D9 — Finalize finds open PR, merges without text edit. Purpose met.
- D10 — halted Finalize, `lym loom goto Beam-Await` with the ready record standing, "the dobby lands the beam again". **Broken** (F1).
- D11 — (a) crash between the accept's removals and Done, re-entry, dobby restarted, "the dobby writes the land request again". **Broken** (F1). (b) Second loop exit at Beam-Await days later. **Broken** (F3).
- D12 — halted Finalize, `lym beam start`. Purpose met.
- D13 — run at PR-Review with an open PR, new binary steps, move. Purpose met.
- D14 — review-only; not walked.

**Findings:**

- **F1 — BLOCKING — design — D11 (entry re-start) and D10 (goto bound).** The dobby never rewrites the land request: it keeps a land-episode record in its state file that survives a restart, and `landBeam` returns at once while that record says acted-not-refused and the reading still passes; the record is cleared only when a reading stops passing. In the D11 crash case the ready record stands and nothing works under the beam, so the reading still passes, the restarted dobby skips the request, and Beam-Await waits forever with no notice. Same after D10's goto, so its bound "the dobby lands the beam again once nothing works under it" is false. Evidence: `internal/dobbyengine/land.go:38-50` (`if record.Land != nil && !record.Land.Refused { return nil }`, cleared only at `:46` when `!reading.Passes`), `land.go:79-83` ("so neither this dobby nor a restarted one acts on it again"), `internal/dobbyengine/state.go:43`.

- **F2 — MEDIUM — design — D8, recorded-PR bullet.** A merge refused for a pending required check or a dismissed review matches no named halt. Finalize merges the parent in and pushes the beam branch, giving a new head whose required checks have not run (or whose approval GitHub dismisses as stale); GitHub's refusal then classifies as `RefusalOther`, not `RefusalPending` (which covers only "base branch was modified" / "mergeability … still being computed"), so D8's "a pending merge past the retry window halts naming the check" never fires for a check, and the bullet names no halt for `RefusalOther` — today that refusal's way forward is the generic one naming `lym batten cancel`, wrong for a beam. Evidence: `internal/pullrequest/client.go:316-326` (`classifyRefusal`), `client.go:57-59`, `internal/landingshed/doc.go:266`, `internal/landingshed/finalize_pr.go:296`.

- **F3 — MEDIUM — design — D11, repair cap.** Nothing in the mechanism makes a loop exit "after a wake that completed" a new stop: the repair count is the restep settle records keyed by the stop key (current producer, history length, state), and each loop exit at Beam-Await leaves the same key (same row, same history length, the failed state the dead-loop teardown writes), so the count keeps growing across days. The record names no fact that tells a completed wake apart and no bound on re-steps once every exit counts as new, so a step crash-looping after one wake re-steps without limit. Evidence: `internal/sheddriver/driverules.go:17-18`, `:135-142`, `:201-210`, `:244`; `internal/sheddriver/drive.go:727`; `internal/shedverbs/doc.go:127`.

- **F4 — LOW — design — D7 / D5.** The notice's prescribed sequence overwrites the orch's hand-edited description: `lym beam describe` always regenerates `summary.md` from the commits, so any wording the orch or operator edited into it (the #56 case) is lost at every member landing and `pr update` then pushes that generated text over the PR body. Evidence: decision-record D5 (line 82), D7 (line 112); `internal/landingshed/beamdescribe.go:71`.

- **F5 — LOW — consistency — D11.** "The arbiter's eligibility … extends from tapestry, darn and sampler to pawl" misstates the mechanism: `Eligibility` reads no recipe; a pawl interrupted stop is refused today only because Beam-Await has no `InterruptPolicies` entry (empty policy → hands back). D11's policy change alone makes it eligible; the record names no other change. Evidence: `internal/arbiterstop/arbiterstop.go:80-87`, `:163`; `internal/loomshed/interruptpolicy.go:32-80`.

**Tool calls:**
1. Read `client.go` around `classifyRefusal`, the Beam-Describe writer fields, and dobby `Land` writers — my context lacked the refusal classification behind D8's halts, the writer behind D5's claim, and who clears the dobby's land episode.
2. Grep `dobbyengine` for any reset of the dobby state's land record on start — my context lacked whether a restarted dobby forgets an acted episode.
