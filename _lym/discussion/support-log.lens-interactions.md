```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked the pairs of decisions in the pawl-orch-landing record that act on the same state.

**Pairs checked**
- D8 × D6: broken (F1).
- D2 × D3: broken (F2).
- D3 × D6: broken (F2).
- D4 × D7, D13 × D7: broken (F3).
- D2 × D11: broken (F4).
- D2 × D13: partly broken (F5).
- D8 × D6, closed-record halt: weak (F6).
- D1 × D13 (retired rows against the removed rows): held.
- D3 refusal 5 × D8 (one resolver for a beam): held.
- D3 refusal 4 × D5 × D8 (the description that Finalize's commit reads): held.
- D9 × D6 (`open` adopts without editing, `update` edits, Finalize never edits): held.
- D10 × D13 (a leftover reopen file is unread and harmless): held.
- D12 × D2 and D12 × D13 (a held beam starts the orch only; a moved run is not held, so start brings both): held.
- D7 × D2 and D7 × D12 (no notices while the dobby is down during a landing; Finalize removes the record): held.
- D11 × D13 (a halted Beam-Await is not "interrupted", so it is not re-invoked): held.
- D13 × D4 × D9 (the carried record keeps the old text and the orch updates it): held.
- D3 refusal 2 × the D2 bound (a member filed in the window is caught by the re-check): held.
- `PATTERN-pull-request-mode` change × D8 (entry field or beam record): held.

**F1 — BLOCKING — D8 × D6, D3.** Two of Finalize's halts point to a verb that refuses at Finalize.
- D8 halts naming `lym beam pr open <slug>` and `lym loom resume` in two cases: no record with a pull request required, and a direct push the parent refuses as protected.
- D6 says every `lym beam pr` verb refuses a run that is not at Beam-Await.
- D10 and D3 make ready final, so the run is past Beam-Await and the named recovery cannot run. The run is stuck with no lym way forward.
- Fix: let `pr open` act on a run halted at Finalize, or have Finalize open the pull request itself. Either way, D6's bound must say so.

**F2 — MEDIUM — D2 × D3 × D6.** After a failed re-check, the beam lands without D3's checks being run again.
- D2's refuse path keeps the ready record and leaves the run at Beam-Await while a late member runs and lands. The dobby then asks for the landing again.
- The re-check covers only members, pairs, owners and ready. D3's refusals 4–6 (description, required pull request, mergeable now) are never re-read.
- In that window D6's verbs are still accepted at Beam-Await, including `pr close` and `pr update`. The orch can remove the record or push new commits after the "point of no return".
- So D3's guarantee lapses: Finalize meets pending checks or a missing pull request, which are the halts F1 shows have no way forward.
- Fix: D6 refuses (or `close` refuses) once the ready record stands, and the accept's re-check runs refusals 4–6. Or state this as D3's bound.

**F3 — MEDIUM — D4 × D7, D13.** D7 needs a head commit that the record does not hold.
- D7 announces "from the record's recorded head" when the never-tracked marker is missing.
- `pullrequest.Record` holds `Head` as a branch name. `GatedHead` is only set by PR-Review and `openAtOnce`.
- Neither D4 nor D6 says `pr open` records a head commit, and D13's carried record may hold a stale `GatedHead`.
- Fix: D4 names the head field that `pr open` writes, and D13 sets it on carry.

**F4 — MEDIUM — D11 × D2.** D11's "no hand action" claim fails in one crash window.
- The window is a crash after the accept removed the request and the dobby-stop marker, and before Done.
- The re-invoked Beam-Await finds the Finalize hold with no request. It lifts the hold and posts "run `lym beam start`", but the dobby the accept stopped stays down.
- No land request ever comes, and the D11 criterion "reaches Done through the dobby's land request" does not hold.
- Fix: the interrupted-entry path restarts the dobby (as `refuse` does), or the criterion is narrowed.

**F5 — LOW — D2 × D13.** An old binary's mid-accept crash falls between both decisions.
- If the old binary crashed mid-accept, the board status names Parent-Align while shed's row is Beam-Await.
- D13 moves only a shed status naming a retired row, and D2's crash check now looks for Finalize.
- The dobby-stop marker can therefore stay standing with no interrupted notice.
- Fix: D13 also treats a board status naming a retired row, or D2's check accepts the retired names.

**F6 — LOW — D8 × D6.** The closed-record halt has no lym recovery.
- D8 says "reopen it on GitHub, then `lym loom resume`". No lym verb reopens, and the D6 verbs refuse at Finalize.
- The recovery is the operator's alone, in GitHub's interface. The record's halt text should say so explicitly.

Tool calls made: none.
