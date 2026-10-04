---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:99
    summary: D9's bound for a hand-written issue number says what still works but not how a mistyped number is undone, so an issue hidden from intake list and refused by import has no stated way back onto the list
---

### [LOW] D9's bound leaves the recovery from a wrong hand-written number unstated

**Location:** `_lyx/discussion/decision-record.md` D9 Bound (line 99), read with D10's recorded-issue paragraph (line 121-125).
**Issue:** D9 says a number is not checked against GitHub, a recorded number is skipped by `intake list` and makes `import` a no-op, and `intake close` still closes any open issue.
A mistyped number therefore hides a real open issue from `intake list` and makes `import` of it a no-op, and the record never says what brings it back.
The way is `upsert` of `issues`, which replaces the whole list, but only D9's first paragraph implies it.
The bound should state it, since the focus question asks what bounds a hand-written number on `list` and `import`.
**Fix:** add to D9's Bound that a wrong number is corrected by `upsert` of `issues`, after which `intake list` and `import` see the issue again.

## Focus settlements

1. Failed close after `import` (D10, line 123-124; acceptance line 218): consistent and not a defect.
   `close` recovers it, closing any open issue and writing nothing to the board.
   The failure report carries the exact payload with `completed` and a reason naming the entry's slug, and the acceptance criterion names `intake close` with `completed`.
   For a fold, the entry is the target.
2. Fold against D3 (D10, line 112-115): consistent and not a defect.
   A fold adds only configured labels the target lacks, never a type label to a note, and reports the rest as dropped.
   D3 still refuses on an unlisted label already on the target, with the way forward stated.
   D3's other refusals apply through "passes D3's validation like any write".
3. Sequencing and `issues` (D10 line 127, D9): the checks that depend on shared board state (recorded, slug, `into`) run inside the one lock acquisition with the board write.
   The fetch-derived checks (pull request, closed-and-unrecorded) run on the data fetched outside it, which is the only way to avoid a network call under the lock.
   The bound on a hand-written number is only partly stated; see F1.
4. Support log: no text still describes a retry of `import`.
   Interview item 6 and Q9 both state that `close` recovers a failed close.
