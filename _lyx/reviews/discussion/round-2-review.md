---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:83
    summary: D4's bound claims every loom and darn landing passes its review perch, but a zero bounce budget skips both, and the downgrade's let-through list omits that route
    fix: In D4's rationale, add to the What it lets through sentence a stencil change in a loom run whose Webster-Review budget is 0 (webster_max_bounces, or review_max_bounces when that key is null, settable per run), where the Bouncer settles approved at once, and a darn run with darn.yaml review_max_bounces 0, which goes from Darn-Write straight to Merge-In with no Darn-Review. Then reword the What bounds it sentence so it holds only for a perch with a non-zero budget, e.g. a loom landing whose Webster-Review budget is above 0 passes a judged Burler round, and likewise a darn landing whose review_max_bounces is above 0.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:54
    summary: D2 drops the section-scoped marker-placement rows, which D1's first keep (a Go-filled marker's position relative to a heading) keeps, and its reason (exact marker sets pin presence) does not cover position
    fix: Make D1 and D2 agree. Either (a) in D2's marker-position bullet at line 51, add the section-scoped marker rows (e.g. the `{{.twill_directive}}` rows scoped to `## Step 2 — Explore the codebase` in the plan template and `## Step 4 — How to get answers` in the discussion template, and the `{{.decision_record_path}}`/`{{.support_log_path}}` rows scoped to the fence section) to the `markers_test.go` position table, each as marker-inside-heading's-section, and delete "the section-scoped marker-placement rows included, since `markers_test.go`'s exact marker sets pin presence" from line 54; or (b) narrow D1's first keep at line 34 to "relative to another marker or the end of the body" and add to D1's drop sentence that a marker's placement inside a heading's section goes, saying why that position is not Go-read.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:51
    summary: D2 keeps the two kickstart-pack position rows without saying their prose anchors go, so the kept rows still pin prose and contradict the rewording acceptance criterion
    fix: "In D2's marker-position bullet, add that each moved row anchors on markers, headings and the end of the body only, never on prose: the `{{.kickstart_pack}}` row (today `never a slot-busy exit.{{.kickstart_pack}}\\n`) checks the body ends with the marker, and the `{{.kickstart_pack_inline}}` row (today `do not continue to a later card on top of a broken one.{{.kickstart_pack_inline}}\\n\\n## The sandbox rule`) checks the marker precedes the `## The sandbox rule` heading. Then in the acceptance criterion at line 113, add a heading a kept marker-position row anchors on to the list of exceptions, since renaming `## The sandbox rule` fails that kept row by design."
---

### [MEDIUM] D4's bound overstates which landings pass a judged review

**Location:** _lyx/discussion/decision-record.md:83

**Issue:** The rubric's attack-surface rule asks that a downgraded guard state what it lets through and what bounds it.
D4 says "a loom landing passes Webster-Review and a darn landing passes Darn-Review".
`contracts/recipes/darn-recipe.yaml:60` says `internal/loomrecipe` rewrites Darn-Write's `on_done` to Merge-In when `darn.yaml`'s `review_max_bounces` is 0, and the hub's `_board/_lyx/config/darn.yaml` comment says 0 "runs neither the review nor Describe".
`contracts/recipes/loom-recipe.yaml:81-84` says a perch at 0 keeps its route while its Bouncer settles at once, and `loom.yaml`'s `webster_max_bounces` comment says 0 "runs no review round: the perch's Bouncer settles approved at once".
Both budgets are settable per run through `run_settings`.
So a zero-budget run lands a stencil change with no reviewer reading the new D4 bullet, and the downgrade lets that change through without the bound saying so.
The readers D4 names exist (`rubric_stencil: loom-rubric-webster-review` on the Webster rows, `loom-rubric-darn-review` on the Darn rows), and the orch-pushed fix and operator commit on main are correct let-through routes; only the zero-budget route is missing.

### [MEDIUM] D2 drops section-scoped marker rows that D1 keeps

**Location:** _lyx/discussion/decision-record.md:54

**Issue:** D1's first keep (line 34) keeps "the position of a Go-filled marker relative to another marker, a heading, or the end of the body".
`claims_test.go` holds marker rows scoped by `section:` to a heading, e.g. line 280 (`{{.twill_directive}}` in `## Step 2 — Explore the codebase`, why: the lens review spawns once Step 2's reading is done), line 329 (`{{.twill_directive}}` in `## Step 4 — How to get answers`) and lines 324-325 (`{{.decision_record_path}}`, `{{.support_log_path}}` in the fence section).
Each pins a Go-filled marker's position relative to a heading, which D1 keeps, yet D2 line 54 drops them.
Its stated reason, that `markers_test.go`'s exact marker sets pin presence, covers presence only: moving `{{.twill_directive}}` from Step 2 to Step 5 keeps the marker set intact and fails nothing, so the acceptance criterion "moving a position-pinned marker ... still fails a test" depends on which reading the plan writer takes.

### [MEDIUM] Kept kickstart rows still anchor on prose

**Location:** _lyx/discussion/decision-record.md:51

**Issue:** The two rows D2 keeps are `claims_test.go:580-583`: `never a slot-busy exit.{{.kickstart_pack}}\n` and `do not continue to a later card on top of a broken one.{{.kickstart_pack_inline}}\n\n## The sandbox rule`.
Each pins the marker against the last words of a prose sentence.
D1's first keep allows a position relative to a marker, a heading or the end of the body, not prose, and D2 moves the rows without saying the prose anchor is replaced.
Moved as written, rewording "never a slot-busy exit" or the closing paragraph fails a kept test, contradicting the acceptance criterion at line 113, whose exceptions name no prose next to a marker.
The `## The sandbox rule` row also pins a heading, which that criterion's exception list does not name either.

## Settled without a finding

- Focus 2: D2 line 53 states that the Go-value table test and the twill host-neutral scan carry `//lyx:guard`, and `internal/impactset/guards.go`'s `FindGuardTests` runs every marked top-level test by name in its own package, so a rename of `shuttleengine.MessageTail`, `shedrun.ParkMarkerFileName` or `gatecli.SlotBusyExit` outside `contracts/stencils` fails the renaming card's gate. Not a defect.
- Focus 4: the rubrics under `contracts/stencils/loom/` with a no-coverage item are webster-review (line 85), darn-review (line 66) and merge-review (line 59), and D3 exempts stencil prose in all three; plan-review's Test economy bullet has no such item and D3 adds its own. The literal key rows (`class`, `first_card`, `claims_test.go:154-155,181`) have a disposition: their Go definitions (`loomshed.roundRecord`'s tags in `prrework.go:84-85`, `planparser`'s `first_card` tag in `parse.go:93`) are unexported, so D1's "no Go definition the test can reference" drops them. Not a defect.
