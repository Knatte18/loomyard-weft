```yaml
lens: fix-proposal
outcome: reported
```

**fix-proposal lens: challenges to the proposals the fixer applied as written**

Proposals checked: F1, F2, F3, F5, F6, F7, F8, F9 and F10 were applied as proposed, and F4 was applied with altered wording.

1. **MEDIUM, F8, D9 fourth bullet: "and never again once its first shared-spot card is in".**
   - **Summary:** A task with no shared spots never gets a shared-spot card, so the chair may rejoin after writing the overview's framing paragraph and `## PATTERN changes`, and that rejoin deletes them.
   - **Evidence:** D3 allows a parts file with no shared spots, since only a part is required.
   - D8: `join-parts` "writes `00-overview.md` … and replaces any card files the plan directory held".
   - D9 orders the steps: join, then shared spots, then "writes the overview's framing paragraph and `## PATTERN changes`".
   - D8 Bound: "a rerun discards whatever the chair added after the last join, which D9 keeps the chair from doing."
   - With zero spots the new wording no longer gives that guarantee.
   - The old wording, "after the next step begins", covered this case, even if ambiguously.
   - D9 line "A seam found after the shared-spot cards are in…" has the same gap.
   - A bound that holds whether or not the plan has shared spots: "never again once it has written anything into the plan directory after the join".

2. **MEDIUM, F3, D5: the appended sentence about a probe-removed done writer.**
   - **Summary:** The appended line contradicts the earlier sentence it should have replaced, so the chair is told two opposite things about a part D9 now says is still open.
   - **Evidence:** D5 still says "counts as finished … the chair is told its part stands and the seat no longer answers".
   - The appended sentence says the chair is told "the writer was stopped at the interruption and that any fix the chair had asked of it is lost".
   - D9 now says "a part whose fix request was outstanding at a resume is still open".
   - The D5 acceptance criterion keeps both: "tells the chair its part stands; a resume after a done writer was stopped mid-fix tells the chair the fix was lost".
   - The review's fix asked for the line the chair is told to say this, which means one line, not two.

3. **MEDIUM, F4, D2: "whose default in `tapestry.yaml` is twice `plan_timeout_min`'s default".**
   - **Summary:** The fixed literal brings F4 back whenever `plan_timeout_min` is raised: the chair times out at or before a writer.
   - **Evidence:** `plan_timeout_min` is hub-tunable and settable per profile (`internal/loomengine/tapestry.go:33`, `:97`; `tapestry.yaml:2` default 120).
   - Concrete case: a profile sets `plan_timeout_min: 300`, so the chair keeps the default 240 and each writer gets 300.
   - A slow writer then outlives the chair, and `join` takes `chairDone` without setting `EndedStep`.
   - The new D6 rule then applies: "a chair timeout with no writer ended maps as `Plan-Write`'s chair timeout does today".
   - The result is an uncommitted error naming the chair, the outcome F4 was raised to prevent.
   - The review said "default twice plan_timeout_min", which can be read as derived.
   - Fixes: derive the default from the run's effective `plan_timeout_min` when the key is unset, or refuse at config load a `parts_chair_timeout_min` not above `plan_timeout_min`.

4. **LOW, F1, D8: way forward "goto --to Parts-Plan-Write, then lym loom resume, whose fresh spawn archives every part, rotates the plan directory and has the new chair rejoin".**
   - **Summary:** For a run paused mid-row with a live chair, there is no fresh spawn, so the stated outcome is wrong.
   - **Evidence:** `Goto` admits the current row and writes paused (`internal/shedengine/goto.go:68-140`).
   - Under reinvoke (D13), the next step probes, and with a live chair `Resume` attaches instead of running `prepareFreshSpawn` (D6 Bound; `resume.go` `Probe` doc).
   - No parts are archived, and the existing chair carries on and runs the join itself.
   - That still makes progress, but the claimed archive, rotation and "new chair" do not happen for a paused run.

Proposals checked and kept:
- F2: the Stuck mapping for a writer timeout, set against the chair-timeout error.
- F5: the `validate-parts` refusal.
- F11: the ban on part writers running `insert-card`. `insert-card` works only on the plan directory, so the ban costs a writer nothing.
- F6, F7, F9 and F10.

None of these gave a concrete breaking case.

Tool calls:
- grep of `internal/shedengine`/`loomengine` for goto rules: needed to know whether goto admits the current row; not in context.
- `sed` of `internal/shedengine/goto.go` lines 55 to 165: the goto write state and the target check; not in context.
- grep for `insert-card` scope and `plan_timeout_min` tunability: needed to test the F1 ban and the F4 default; not in context.
